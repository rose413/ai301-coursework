# Plan: issue #38, integration tests for authentication edge cases

**Issue:** [codepath/pathreview-ai301-fa26-s3#38](https://github.com/codepath/pathreview-ai301-fa26-s3/issues/38)
**Builds on:** my repro comment, [#issuecomment-5864870840](https://github.com/codepath/pathreview-ai301-fa26-s3/issues/38#issuecomment-5864870840) (`main` @ `2f4e82f`, Python 3.11.3, Windows 11)
**Branch:** `fix/38-auth-middleware-tests` on my fork (the course house rule is `fix/<issue-number>-<slug>`)

The issue asks for tests that call a protected endpoint with four inputs:
an expired token, a malformed token, no `Authorization` header, and a token
signed with a different secret. `api/middleware/auth.py` has no coverage, and
`tests/integration/` holds only an empty `__init__.py`.

## Repro evidence I'm relying on

All quotes are from my posted repro comment.

Premise check:

> - `tests/integration/` contains only `__init__.py`, and it is 0 bytes.
> - No file under `tests/` references `get_current_user` or `api/middleware/auth`.
> - `core/security.py` *is* covered — `tests/unit/test_security.py` has 25 tests, including tampering and malformed input — but none of them exercise an **expired** token, and none go through the middleware.

Steps: save `repro_38.py` (a minimal FastAPI app with one `/protected` route
that depends on `get_current_user`, with `get_db` overridden by a stub session
that records each `execute` call) in the repo root, then run
`python repro_38.py`. Observed:

```
scenario                 status  detail                               WWW-Auth  db?
----------------------------------------------------------------------------------------
valid token (control)    200     {'user_id': '11111111-1111-1111-1111 -         yes
no Authorization header  401     Not authenticated                    Bearer    no
malformed token          401     Invalid authentication credentials   Bearer    no
expired token            401     Invalid authentication credentials   Bearer    no
wrong signing secret     401     Invalid authentication credentials   Bearer    no
```

Finding 1:

> `api/middleware/auth.py:44-49` raises `detail="Token has expired"`, but an expired token never gets there. [...] `jwt.decode` raises `ExpiredSignatureError`, which is a `JWTError` subclass, so `decode_access_token` returns `None` and the payload-is-None check fires before line 44 is ever reached.

Finding 2:

> The stub session was never called in any of the four failure scenarios (`db?` = no); only the valid-token control reaches it. All four fail during token decode, before the `select(User)` at `auth.py:59`.

> `DATABASE_URL` can't just be pointed at SQLite to sidestep Postgres. `core/database.py:11-18` hardcodes `pool_size=10, max_overflow=20`, which SQLite's `StaticPool` rejects at import

The repro's limit, which this plan has to close:

> I also have not yet run any of this through the real route modules in `api/routes/`; the protected endpoint above is a minimal stand-in.

## Diagnosis

This is missing coverage, not a runtime bug. The middleware already does what
the issue's four cases need: every failure input returns 401 with
`WWW-Authenticate: Bearer` (the table above). Nothing in the suite would notice
if that stopped being true, because no test imports `api/middleware/auth.py`.

The repro also shapes how the tests have to be written:

1. **The four failure cases are decided before the database is touched**
   (`db?` = no on every failure row). The tests need no Postgres. They do need
   `get_db` overridden, because SQLite isn't an option (the `pool_size` error
   above) and a real session would need a server.
2. **The expired token comes back with the generic detail, not "Token has
   expired"** (Finding 1). A test asserting the message from the source would
   fail. This is a real defect, a dead branch at `auth.py:43-49`, but it isn't
   what #38 asks for, and fixing it would change runtime code in a test-only
   issue.
3. **The 401 for a missing header comes from `OAuth2PasswordBearer`, not from
   `get_current_user`.** Its detail is `Not authenticated`, which is different
   from the other three. So the tests check the detail per case instead of one
   shared message.

## Scope

**In scope (one change: a new integration test module plus its fixtures)**

- The four cases the issue names, each through a real protected route of the
  real app, `GET /profiles/{profile_id}` (`api/routes/profiles.py:110`) on
  `api.main.app`. This closes the repro's "minimal stand-in" gap and shows the
  middleware is actually wired to the routes.
- One valid-token control test. Without it, a fixture that broke every
  request would make the four 401 assertions pass for the wrong reason. The
  repro's control row is what showed the stub endpoint authenticates.
- A check that the database stub was never called in the four failure cases,
  carried over from the repro's `db?` column.

**Not in scope (stated so nobody expects it in the PR)**

- `api/middleware/auth.py` and `core/security.py`: no edits. That includes the
  unreachable `"Token has expired"` branch (Finding 1) and the
  `datetime.utcfromtimestamp` deprecation at `auth.py:44`. I'll describe the
  dead branch in a separate issue rather than fix it here.
- `core/database.py`: the hardcoded pool args stay. The `get_db` override
  avoids them, so changing them isn't needed.
- `api/main.py`: the startup hook re-raises at line 92 when the DB is down. I
  work around it in the fixture (see Approach) instead of changing it.
- The `user is None` (deleted user) and DB-error 500 paths at
  `auth.py:59-69`. My repro didn't exercise them, and the issue doesn't ask
  for them. I'm leaving them for later, deliberately.
- `tests/conftest.py`: the issue lists it, but I'm not editing it. The new
  fixtures go in `tests/integration/conftest.py` so the root conftest and the
  unit suite stay as they are (I said so in the repro comment, and nobody
  objected).
- The `integration` marker's description in `pyproject.toml:180` ("require
  Docker services") stays as written, even though these tests need no
  services. Rewording a shared marker is a separate decision.
- `.github/workflows/ci.yml`: the "exit code 5" shim comment says to delete
  the shim once the directory holds a test. That's CI config, not test code,
  so I'll point it out in the PR rather than change it here.

## Files

| File | Change |
|---|---|
| `tests/integration/conftest.py` | **new**: fixtures `db_stub`, `client`, `make_token` |
| `tests/integration/test_auth_middleware.py` | **new**: 5 tests (4 cases + 1 control) |

Nothing else changes.

## Approach

1. **`tests/integration/conftest.py`**
   - `StubSession`: an `async execute(stmt)` that appends to a `calls` list
     and returns a `StubResult`. The result's `scalar_one_or_none()` returns a
     `User` with a fixed UUID `id` (the user lookup at `auth.py:59-60`). Its
     `scalars().first()` returns `None` (the profile lookup in
     `core/services/profile_service.py:46-47`). This is the repro's stub, with
     one method added so the real route can run past auth.
   - `db_stub` fixture: a fresh `StubSession` per test.
   - `client` fixture: sets `app.dependency_overrides[get_db]` to an async
     generator that yields `db_stub`, returns `TestClient(app)` **without** a
     `with` block, and clears the overrides when the test ends. Without `with`,
     Starlette doesn't run the startup event, so `init_db()` never tries
     Postgres (`api/main.py:84-92`).
   - `make_token(secret=settings.secret_key, delta=timedelta(minutes=30))`:
     `jose.jwt.encode({"sub": USER_ID, "exp": datetime.now(UTC) + delta}, secret, algorithm=settings.jwt_algorithm)`.
     This is the same construction as the repro's `token()`. It reads from
     `settings`, so it follows whatever the app is configured with.
   - All of these get Google-style docstrings, as CONTRIBUTING asks.
2. **`tests/integration/test_auth_middleware.py`**
   - Module-level `pytestmark = pytest.mark.integration`, so
     `make test-integration` (`pytest tests/integration -v -m integration`)
     selects them. Unmarked tests would be skipped by that target.
   - `PROFILE_URL = "/profiles/22222222-2222-2222-2222-222222222222"`.
   - `test_missing_authorization_header`: no header → `401`,
     `detail == "Not authenticated"`, `WWW-Authenticate == "Bearer"`,
     `db_stub.calls == []`.
   - `test_malformed_token`: `Bearer not.a.jwt` → `401`,
     `detail == "Invalid authentication credentials"`, `Bearer`, no DB calls.
   - `test_wrong_signing_secret`: `make_token(secret="other-secret")` → same
     as malformed.
   - `test_expired_token`: `make_token(delta=timedelta(minutes=-5))` → `401`,
     `Bearer`, no DB calls. **No assertion on `detail`.** Asserting `"Token has
     expired"` would fail today. Asserting the generic message would lock the
     dead branch in, so whoever fixes Finding 1 would have to delete a test that
     looked correct. A comment on the test will point to the follow-up issue.
   - `test_valid_token_reaches_route` (control): `make_token()` → `404`,
     `detail == "Profile not found"`, `len(db_stub.calls) == 2` (the user
     lookup, then the profile lookup). A 404 rather than 401 shows auth passed
     and the request reached the route handler.
3. Run `make lint`, `make typecheck`, `make test-unit`, `make test-integration`
   locally, then push to the fork.
4. Open the follow-up issue for the unreachable `"Token has expired"` branch,
   citing Finding 1, and link it from the `test_expired_token` comment.

## Test plan

**A. My repro, re-run unchanged.** `python repro_38.py` at the repo root on
the branch. Expected: the same five rows as in my repro comment (200/401/401/401/401,
`db?` yes/no/no/no/no). The change adds tests only, so a different table would
mean something outside `tests/` changed.

**B. The same five scenarios, now as tests through the real app.** Run
`pytest tests/integration -v -m integration` with **no** Postgres or Docker
running.

| Scenario | Repro result (stand-in route) | Expected from the new test (real `GET /profiles/{id}`) |
|---|---|---|
| valid token | 200, DB called | `PASSED`: 404 `Profile not found`, 2 DB calls |
| no header | 401 `Not authenticated`, Bearer, no DB | `PASSED`: same three facts asserted |
| malformed | 401 `Invalid authentication credentials`, Bearer, no DB | `PASSED`: same |
| expired | 401 `Invalid authentication credentials`, Bearer, no DB | `PASSED`: 401, Bearer, no DB (detail not asserted) |
| wrong secret | 401 `Invalid authentication credentials`, Bearer, no DB | `PASSED`: same |

Before the change, the same command prints `collected 0 items` and exits 5.
After it, the command prints `5 passed`. That's the observable difference.

**C. The tests can fail.** Temporarily make two edits, run the command after
each, then revert:
- In `api/middleware/auth.py`, change `if payload is None: raise credentials_exception` to
  `if payload is None: return None`. Expected: `test_malformed_token`,
  `test_wrong_signing_secret`, and `test_expired_token` **fail**, because the
  request gets past auth.
- In the test, change the no-header expected status to `200`. Expected:
  `test_missing_authorization_header` **fails** with `assert 401 == 200`.

If either edit leaves the suite green, the tests aren't checking what they
claim to, and I'll fix the tests before opening a PR.

**D. Nothing else moved.** `make test-unit` gives the same pass/xfail counts as
on `main` (only new files under `tests/integration/`). `make lint` and
`make typecheck` are clean on the two new files.

## Risks and unknowns

- **Driving `api.main.app` without Postgres is unverified by me.** My repro
  used a stand-in app. A classmate's follow-up on the thread reports that the
  startup hook re-raises (`api/main.py:92` is a bare `raise`, which I've read
  and confirmed) and that a `TestClient` used without `with` skips startup. I
  haven't run that myself, because the environment from my repro isn't
  installed on this machine right now. **First build step:** run the
  no-header test alone with Docker stopped. If it errors at setup, the fallback
  is the repro's approach: a test-only `FastAPI()` that mounts
  `api.routes.profiles.router`. That still uses a real route module and still
  avoids the startup hook. I'd record whichever one I end up using under
  Deviations.
- **`RequestIDMiddleware` and the CORS middleware run on these requests.** The
  repro bypassed both. I don't expect either to change a 401, but test B is
  where that would show up.
- **Stub shape vs. the real queries.** The control depends on
  `get_profile` calling `result.scalars().first()`. If the route or service
  reads the result differently, the control fails with a 500 instead of a 404.
  That would be a stub problem, not an auth problem, and I'd fix it in the
  stub.
- **CI runs `pytest tests/integration` without `-m integration`.** Marked
  tests still run there, so this doesn't break anything. But once the
  directory has tests, the exit-5 shim in `ci.yml` is dead code. I'll mention
  it in the PR (see Not in scope).
- **Python 3.12+**: `auth.py:44` uses `datetime.utcfromtimestamp`, which is
  deprecated there. In the four failure cases that line isn't reached, but in
  the control it is, so a `DeprecationWarning` could show up on newer Python.
  CI pins 3.11. I've only run on 3.11.
- **The expired-token detail.** If a maintainer would rather the tests pin the
  current generic message, it's a one-line addition to `test_expired_token`.
  I'm leaving it out by default for the reason in Approach 2.

## Deviations

None. The build matches the plan: the same two new files, the same five tests
with the same assertions, and no edits outside `tests/integration/`. Every
check in the test plan came out as predicted. The posted plan comment is still
accurate, so I haven't posted a correction on the issue.

What the build settled, for the record:

- **The first build step held, so I didn't use the fallback.** With Docker
  stopped, `TestClient(api.main.app)` without `with` returned 401 on
  `GET /profiles/{id}`. Entering `with TestClient(app)` raised
  `ConnectionRefusedError` from the startup hook, as reported on the thread.
  The tests drive the real `api.main.app`, not a test-only `FastAPI()`.
- **A.** `repro_38.py` (copied verbatim from my repro comment, run with
  `PYTHONPATH=.`) printed the same five rows: 200/401/401/401/401, `db?`
  yes/no/no/no/no.
- **B.** `pytest tests/integration -v -m integration` went from
  `collected 0 items` to `5 passed`. Plain `pytest tests/integration`, which is
  what CI runs, also gives `5 passed` with exit 0.
- **C.** Changing `auth.py:36` to `return None` failed exactly
  `test_malformed_token`, `test_wrong_signing_secret`, and `test_expired_token`.
  Changing the no-header expected status to 200 failed with `assert 401 == 200`.
  I reverted both, and `git diff api/` is empty.
- **D.** `make test-unit` gives `375 passed, 53 xfailed` with and without the
  new files. `make lint` and `make typecheck` pass, and ruff, black, and mypy
  are clean on the two new files.

Differences in the environment, not the build:

- I set up a fresh `.venv` (`pip install -e ".[dev]"`). It resolved
  **fastapi 0.142.2 / starlette 1.7.0**, while the repro ran on
  0.116.1 / 0.47.3. Python is the same 3.11.3. Run A matched anyway, so the
  version change didn't affect any of these cases.
- The 3 warnings in the test run come from importing existing app code:
  Pydantic class-based config at `core/config.py:7` and FastAPI `on_event`
  deprecations at `api/main.py:84`. None come from the new tests, and the
  `utcfromtimestamp` warning I flagged under Risks didn't show up on 3.11.

Still open (not started yet, as opposed to changed):

- Approach step 4: the follow-up issue for the unreachable `"Token has expired"`
  branch isn't open yet. Until it is, the comment in `test_expired_token` links
  to Finding 1 in my repro comment. I'll swap in the issue link once the issue
  exists.
- Approach step 3's push, and the PR, which should say I used the
  real-app approach and mention the now-dead exit-5 shim in `ci.yml:90`.
