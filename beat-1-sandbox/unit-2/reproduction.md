# Unit 2 — Claim and Reproduce

Path: `beat-1-sandbox/unit-2/reproduction.md`

Record of your claim and reproduction on the issue you chose in Unit 1, and of the
evaluation runs that produced `eval-run.txt`. This file is graded at the path above; a copy
kept anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your identity upstream

**GitHub username**

[Your GitHub username, exactly as it appears on your profile — no `@`, no profile URL. Your
comments upstream are identified by this name.]
rose413

---

## Posted upstream

**Claim comment**

[Link to the comment where you claimed the issue. Use the comment's own permalink, not the
issue page on its own. **Then paste the text of that comment underneath the link** — the
pasted text is what this field is graded on, so copy across what you actually posted.]

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/38#issuecomment-5864176862

Picking this up: `api/middleware/auth.py` has no test coverage and
`tests/integration/` holds only an empty `__init__.py`, as reported
here. I have not confirmed the gap on a fresh clone yet — my next step
is running the suite with coverage on current `main` to record where
`auth.py` stands today. I'll post a short report with my environment,
the exact commands, and the coverage output, then add the four cases
the issue names: expired token, malformed token, no `Authorization`
header, and a token signed with a different secret. Third contribution
for me, so flagging that.

**Reproduction comment**

[Link to the comment where you posted your reproduction. It must record the environment
(OS, relevant versions, code state), steps a stranger could follow, and what you observed.
**Then paste the text of that comment underneath the link** — the pasted text is what this
field is graded on, so copy across what you actually posted.]

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/38#issuecomment-5864870840

Following up on my claim above. I said I'd post coverage output — the coverage question turned out to be answerable by inspection instead, since nothing under `tests/` touches this module at any level, so the figure is zero and the useful work was elsewhere. What I did instead was pin down what the middleware actually does today, so the new tests assert observed behavior rather than behavior inferred from reading the source.

This is a test-coverage request rather than a bug report, so there's no failure to reproduce as such. What follows is (a) verification that the coverage gap is real and (b) a baseline run of the four scenarios named in the issue. Two things turned up that will shape the fixtures.

## Environment

| | |
|---|---|
| OS | Windows 11 Home 10.0.26200 |
| Python | 3.11.3 |
| Repo state | `main` @ `2f4e82f`, working tree clean |
| Database | none running — not needed, see finding 2 |
| Config | `core/config.py` defaults, no `.env` (`secret_key='dev-secret-key-change-in-production'`, `jwt_algorithm=HS256`) |

Package versions in the run:

```
fastapi 0.116.1           starlette 0.47.3      python-jose 3.5.0
SQLAlchemy 2.0.46         asyncpg 0.31.0        pydantic 2.9.2
pydantic-settings 2.14.1  structlog 26.1.0      passlib 1.7.4
httpx 0.27.0              anyio 4.4.0           pytest 9.1.1
pytest-asyncio 1.4.0
```

## Premise check

Confirmed at `2f4e82f`:

- `tests/integration/` contains only `__init__.py`, and it is 0 bytes.
- No file under `tests/` references `get_current_user` or `api/middleware/auth`.
- `core/security.py` *is* covered — `tests/unit/test_security.py` has 25 tests, including tampering and malformed input — but none of them exercise an **expired** token, and none go through the middleware.

So `api/middleware/auth.py` has no coverage at any level, as the issue says.

## Steps to reproduce

1. Clone at `2f4e82f` and install the runtime deps (`pip install -e ".[dev]"`, or at minimum `fastapi python-jose[cryptography] structlog passlib[bcrypt] sqlalchemy asyncpg pydantic-settings httpx`). `asyncpg` has to be importable because `core/database.py` builds the engine at import time, but no server needs to be running — the engine is never connected.
2. Save this as `repro_38.py` in the repo root:

```python
from datetime import UTC, datetime, timedelta

from fastapi import Depends, FastAPI
from fastapi.testclient import TestClient
from jose import jwt

from api.middleware.auth import get_current_user
from core.config import settings
from core.database import get_db
from core.models.user import User

USER_ID = "11111111-1111-1111-1111-111111111111"
db_calls: list[str] = []


class StubResult:
    def scalar_one_or_none(self):
        u = User()
        u.id = USER_ID
        u.email = "jane@example.com"
        return u


class StubSession:
    async def execute(self, _stmt):
        db_calls.append("execute")
        return StubResult()


async def override_get_db():
    yield StubSession()


app = FastAPI()
app.dependency_overrides[get_db] = override_get_db


@app.get("/protected")
async def protected(current_user: User = Depends(get_current_user)):
    return {"user_id": current_user.id}


client = TestClient(app)


def token(secret: str, delta: timedelta) -> str:
    return jwt.encode(
        {"sub": USER_ID, "exp": datetime.now(UTC) + delta},
        secret,
        algorithm=settings.jwt_algorithm,
    )


SECRET = settings.secret_key
cases = [
    ("valid token (control)", {"Authorization": f"Bearer {token(SECRET, timedelta(minutes=30))}"}),
    ("no Authorization header", {}),
    ("malformed token", {"Authorization": "Bearer not.a.jwt"}),
    ("expired token", {"Authorization": f"Bearer {token(SECRET, timedelta(minutes=-5))}"}),
    ("wrong signing secret", {"Authorization": f"Bearer {token('other-secret', timedelta(minutes=30))}"}),
]

print(f"{'scenario':<24} {'status':<7} {'detail':<36} {'WWW-Auth':<9} db?")
print("-" * 88)
for name, headers in cases:
    db_calls.clear()
    r = client.get("/protected", headers=headers)
    detail = str(r.json().get("detail", r.json()))
    www = r.headers.get("www-authenticate", "-")
    print(f"{name:<24} {r.status_code:<7} {detail[:36]:<36} {www:<9} {'yes' if db_calls else 'no'}")
```

3. Run it from the repo root: `python repro_38.py`. No `PYTHONPATH` is needed — the script's own directory is what goes on `sys.path`, so keeping it in the root is what makes the `api.` and `core.` imports resolve.

The DB session is stubbed, so the run needs no Postgres, and the `db?` column records whether the middleware reached the database at all. The first row is a control: a valid token, to show the endpoint does authenticate normally.

## Observed

```
scenario                 status  detail                               WWW-Auth  db?
----------------------------------------------------------------------------------------
valid token (control)    200     {'user_id': '11111111-1111-1111-1111 -         yes
no Authorization header  401     Not authenticated                    Bearer    no
malformed token          401     Invalid authentication credentials   Bearer    no
expired token            401     Invalid authentication credentials   Bearer    no
wrong signing secret     401     Invalid authentication credentials   Bearer    no
```

All four failure cases return 401 with `WWW-Authenticate: Bearer`, which matches the docstring. Two details are not what the source reads like it intends.

### Finding 1 — the expired-token branch is unreachable

`api/middleware/auth.py:44-49` raises `detail="Token has expired"`, but an expired token never gets there. python-jose validates `exp` inside `jwt.decode`, and `decode_access_token` catches every `JWTError` and returns `None` (`core/security.py:85-86`), so an expired token exits at `auth.py:35-36` as the generic `credentials_exception`. The expired-token row above is that exit: `jwt.decode` raises `ExpiredSignatureError`, which is a `JWTError` subclass, so `decode_access_token` returns `None` and the payload-is-None check fires before line 44 is ever reached.

A test written from the source that asserts `detail == "Token has expired"` will fail. Worth deciding which way to resolve it before the tests go in: either they lock in the current generic 401, or the unreachable branch is treated as a separate bug. I'd lean toward asserting current behavior and opening a follow-up issue for the dead branch, so this one stays test-only — happy to go the other way if you'd prefer.

### Finding 2 — these cases never touch the database

The stub session was never called in any of the four failure scenarios (`db?` = no); only the valid-token control reaches it. All four fail during token decode, before the `select(User)` at `auth.py:59`.

So these tests need no Postgres and no Docker. That's worth flagging because `pyproject.toml:180` defines the `integration` marker as "Integration tests (require Docker services)". They'd live in `tests/integration/` per the issue but can run in CI with no services — or they could be marked `unit`. Let me know which you'd prefer.

One related snag for fixture design: `DATABASE_URL` can't just be pointed at SQLite to sidestep Postgres. `core/database.py:11-18` hardcodes `pool_size=10, max_overflow=20`, which SQLite's `StaticPool` rejects at import:

```
TypeError: Invalid argument(s) 'pool_size','max_overflow' sent to create_engine(),
using configuration SQLiteDialect_aiosqlite/StaticPool/Engine.
```

So the fixture either keeps `asyncpg` installed and overrides `get_db` as above (what I did — no server required), or `core/database.py` needs those pool args made conditional. The first keeps this issue test-only, so that's the route I'd take.

`tests/conftest.py` currently holds only the two sample-text fixtures, so the app/client/session fixtures would be new. I'd put them in `tests/integration/conftest.py` rather than the root one to leave the unit tests untouched, unless you'd rather they be shared.

## What I did not check

Only the four scenarios named in the issue, plus the valid-token control. I did not test against a real Postgres, did not exercise the `user is None` (deleted user) or the DB-error 500 path in `auth.py:61-69`, and ran only on Python 3.11 — note `auth.py:44` uses `datetime.utcfromtimestamp`, which is deprecated on 3.12+, so it may warn on newer interpreters than the one I used. I also have not yet run any of this through the real route modules in `api/routes/`; the protected endpoint above is a minimal stand-in.



## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

[The agreement score of each run you did, in order. A single run is a complete answer if
only one run occurred. **The last score in your list must match the agreement line in the
`eval-run.txt` you committed** — that file is the record of your final run.]
Single run: 18/20 agreement (clear-accept 6/8, disclosure 1/1, no-evidence 4/4,
unfollowable-comms 3/3, wrong-target 4/4), bar 18/20: PASS. This is the only full run I
did, and it matches the agreement line in the committed `eval-run.txt`.


**Package analysis**

[Pick one scored package (`pkg-01` through `pkg-20` — the four `calib-` packages are never
scored). Name it by id, say what your rubric decided and what the gold label said, and
explain why your rubric read it that way.]
`pkg-10` (starship/starship#7648, "Prompt disappears when path is a symlink to a
subdirectory inside a Git repository"). My rubric's verdict was reject, failing
`target-faithful`; the gold label was accept, with the note "honest cannot-reproduce:
exact layout and config, prompt artifact shown, names the environment differences
(Linux+zsh vs macOS+fish) and the PWD-resolution hypothesis for why fish matters."
`control-run` also graded fail, but it is `preferred` and never gates the verdict, so
`target-faithful` is what decided this one.

Why my rubric read it that way: the package is a cannot-reproduce. The issue names macOS
with fish 4.7.1 as the environment in which the `directory` module vanishes, and the
report ran Ubuntu 24.04 with zsh 5.9. My `target-faithful` check has two clauses, and the
package splits them. Clause (b) is satisfied outright — the report names the delta in its
own words ("The report is macOS + fish 4.7.1; shell and OS both differ, starship version
matches"), which is exactly what that clause asks for. Clause (a) is what failed: it
requires that "the run preserves the condition the issue names as producing the failure,"
and the report's own closing paragraph concedes the condition was not preserved — "A fish
shell resolving `PWD` logically looks necessary to hit the `contract_repo_path` failure;
I did not have one available for this attempt." Read literally, a different code path was
exercised, so clause (a) fails, and one failed required check is a reject.

The gold label reads the same evidence as the package's strength rather than its defect:
the report ran the exact layout and config from the issue, showed the prompt artifact,
and correctly identified *why* its environment could not trigger the bug. That is a
faithful negative result, which my own verdict rule says explicitly should pass —
"Reporting a negative result faithfully is a contribution." My two components disagree
with each other here, and `target-faithful` won because it is a required check while that
sentence sits in the prose under the verdict rule rather than in a check.


**Check rationale**

[Quote one check from the `rubric.md` you uploaded to `tools/repro-check/`, exactly as it reads now.
Then say why it reads that way — what you revised to get there, or what you rejected in
favour of it.]
> Both must hold. (a) Trigger: the run preserves the condition the issue names as causing
> the failure — the same input shape, syntax, and precondition (exactly one custom header,
> the offset-from-end range, the tuple passed to the same call). Changing *how* the run is
> performed while preserving that condition is fine — running offline, scripting the setup,
> a fresh playground — especially when the report says so. What fails is changing the
> condition itself, so that a different code path is exercised: a swapped operator, an
> altered expression, a skipped precondition. (b) Version: the version or build tested is
> the one the issue targets, or the report names the difference in its own words. Fail when
> an older or newer version is tested and the delta is never called out.

(`target-faithful`, from `rubric.md`.) This check exists to catch the package that looks
like a reproduction but is actually reproducing something else — the adjacent symptom, the
simplified input that no longer hits the reported path. The `wrong-target` category is 4/4
for me, so the check does the job it was written for.

The long middle sentence is the part I deliberately wrote in, and it is the reason the
check reads the way it does: without it, the check would reject any report that scripted
its setup, ran offline, or used a fresh playground instead of the reporter's exact
workflow. Those are all legitimate — a stranger reproducing an issue should be free to
change *how* they run it. So I separated method from condition and said so explicitly,
rejecting the simpler "the run must match the issue's steps" phrasing in favour of a rule
about which code path gets exercised. Clause (b) is separate because a version mismatch is
not a trigger problem; it only becomes a defect when the report hides it, which is why the
escape hatch is "or the report names the difference in its own words."


**Trade-offs**

[Every check gives something up. Any one of these is a complete answer: a package whose
result it changes, a canary you re-ran with `--only`, a case you accept it will miss, or a
stated reason nothing changed elsewhere. "Nothing changed, and here is how I know" earns
the point in full when the reason follows.]
What `target-faithful` gives up is the honest cannot-reproduce, and `pkg-10` is the
package whose result it changes. The two clauses were both written with a *successful*
reproduction in mind, where preserving the trigger is the whole point. On a
cannot-reproduce, the environment necessarily differs — that difference is the finding, not
a flaw — so clause (a) fails by construction on exactly the reports my verdict rule says
should pass. The check cannot distinguish "I changed the condition and did not notice"
from "I could not obtain the condition, and here is which part I was missing and why it
matters," because both descriptions are true of a run that did not preserve the trigger.

Both of my two misses sit in `clear-accept` (6/8), and neither is in a category I read
blind — `wrong-target`, `unfollowable-comms`, `no-evidence`, and `disclosure` are all full
marks. So the failure mode is one-directional: my rubric is strict rather than blind, and
it costs me accepts rather than letting bad packages through. For a rubric whose job is
deciding whether *my own* draft is ready to post, I would rather it hold a package I
should have sent than wave through one I should not have, so I am accepting this miss
rather than loosening the check.

The fix, if I wanted one, belongs in the check and not in the run: clause (a) would need a
carve-out making it inapplicable when the report's stated result is a cannot-reproduce, so
that such packages are decided by `artifact-backs-claim` (which already handles them — "An
artifact showing that nothing went wrong passes when the report's claim is that it could
not reproduce") and by clause (b) alone. I did not make that edit, because at 18/20 with
the category floor met the run already clears the bar, and re-running to chase two
clear-accepts would invalidate the committed `eval-run.txt` without changing what I learned
from it.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/repro-check/`.
