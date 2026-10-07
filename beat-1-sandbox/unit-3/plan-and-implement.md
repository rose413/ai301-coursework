# Unit 3 — Plan and Build

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

Record of your plan, the branch you built it on, and the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in the
repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Posted upstream

**GitHub username**

[Your GitHub username, exactly as it appears on your profile - no @, no
profile URL. Your comment upstream is identified by this name, and it is
the only thing that ties it to you. Several students may plan the same
house issue, so this is what keeps their comments off your score and
yours off theirs.]
rose413

**Plan comment**

[Link to the comment where you posted your plan on the issue. Use the comment's own
permalink. **Then paste the text of that comment underneath the link** — the pasted text is
what this field is graded on, so copy across what you actually posted.]
https://github.com/codepath/pathreview-ai301-fa26-s3/issues/38#issuecomment-6029481998

---

## Your branch

**Branch**

[The name of the branch you built the change on, exactly as it appears in your fork. The
naming shape is a type prefix, then the issue number, then a short description. **The issue
number in the branch name must be the number of the issue you claimed** — a name carrying
any other number does not satisfy this field.]
fix/38-auth-middleware-tests

**Evidence**

[Your Unit 2 reproduction steps re-run against the built change: the before, then the
after. Paste both, including the commands you ran and their output.]
Both runs are on `fix/38-auth-middleware-tests` at `2f4e82f` (the same commit as `main`)
with the two new files in place: `tests/integration/conftest.py` and
`tests/integration/test_auth_middleware.py`. Environment: Windows 11 Home 10.0.26200,
Python 3.11.3, a fresh `.venv` from `pip install -e ".[dev]"` (fastapi 0.142.2,
starlette 1.7.0, python-jose 3.5.0, pytest 9.1.1), with no Postgres and no Docker running.

**A. The Unit 2 repro script, re-run unchanged.** Command, with `repro_38.py` copied
verbatim out of my posted repro comment:

```
PYTHONPATH=. python repro_38.py
```

Before (as posted in Unit 2, on `main` @ `2f4e82f`, fastapi 0.116.1 / starlette 0.47.3):

```
scenario                 status  detail                               WWW-Auth  db?
----------------------------------------------------------------------------------------
valid token (control)    200     {'user_id': '11111111-1111-1111-1111 -         yes
no Authorization header  401     Not authenticated                    Bearer    no
malformed token          401     Invalid authentication credentials   Bearer    no
expired token            401     Invalid authentication credentials   Bearer    no
wrong signing secret     401     Invalid authentication credentials   Bearer    no
```

After (on the branch, fastapi 0.142.2 / starlette 1.7.0):

```
scenario                 status  detail                               WWW-Auth  db?
----------------------------------------------------------------------------------------
valid token (control)    200     {'user_id': '11111111-1111-1111-1111 -         yes
no Authorization header  401     Not authenticated                    Bearer    no
malformed token          401     Invalid authentication credentials   Bearer    no
expired token            401     Invalid authentication credentials   Bearer    no
wrong signing secret     401     Invalid authentication credentials   Bearer    no
```

Identical, which is the expected result here rather than a null one: #38 asks for tests, so
nothing under `api/` or `core/` changed, and a different table would mean the build had
touched runtime code it should not have. The newer FastAPI/Starlette did not move any of
the five cases either.

**B. The same five inputs, now as tests through the real app.** Run A can only show that
the stand-in still behaves the same, because `repro_38.py` builds its own `FastAPI()` app
and never touches the real routes. The build turns those five inputs into tests against the
real `api.main.app` on the real protected route `GET /profiles/{id}`, so this is the
before/after the change actually produces. Command:

```
pytest tests/integration -v -m integration
```

Before (the two new files moved out of `tests/integration/`, leaving what is on `main`):

```
$ ls tests/integration
__init__.py
__pycache__

collecting ... collected 0 items

============================ no tests ran in 0.44s ============================
exit=5
```

After:

```
collecting ... collected 5 items
tests/integration/test_auth_middleware.py::test_missing_authorization_header PASSED [ 20%]
tests/integration/test_auth_middleware.py::test_malformed_token PASSED   [ 40%]
tests/integration/test_auth_middleware.py::test_wrong_signing_secret PASSED [ 60%]
tests/integration/test_auth_middleware.py::test_expired_token PASSED     [ 80%]
tests/integration/test_auth_middleware.py::test_valid_token_reaches_route PASSED [100%]
============================== 5 passed in 1.23s ==============================
exit=0
```

`collected 0 items` / exit 5 became `5 passed` / exit 0. The before run has no check at all,
which is the coverage gap #38 describes. The repro's `db?` column carried across into the
tests as `db_stub.calls == []` on the four failure cases and `len(db_stub.calls) == 2` on
the control.

How the five rows above line up with the five rows in my repro comment:

| Repro input (Unit 2) | Test (Unit 3) |
|---|---|
| valid token (control) | `test_valid_token_reaches_route` |
| no Authorization header | `test_missing_authorization_header` |
| malformed token (`not.a.jwt`) | `test_malformed_token` |
| expired token (`exp` 5 min ago) | `test_expired_token` |
| wrong signing secret (`other-secret`) | `test_wrong_signing_secret` |

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

[The agreement score of each run you did, in order. A single run is a complete answer if
only one run occurred. **The last score in your list must match the agreement line in the
`eval-run.txt` you committed** — that file is the record of your final run.]
Five runs, all on 2026-10-07 (UTC), in order:

1. **No score.** The harness crashed with a Python traceback: a Windows encoding error
   while piping a package to `claude`. 19 of the 20 packages were graded (pkg-02 never
   was), and no agreement line was printed. I turned on Python's UTF-8 mode and ran it
   again.
2. **19/20.** The only miss was pkg-14.
3. **18/20.** The misses were pkg-14 and pkg-03. This was the only time pkg-03 moved: it
   was rejected on `executable`.
4. **19/20.** The only miss was pkg-14.
5. **19/20**, saved with `--save-run`: `agreement: 19/20 scored items  (bar: 18/20: PASS)`
   (clear-accept 6/7, scope-creep 4/4, thread-convention 2/2, unbuildable 3/3,
   wrong-cause 4/4). This is the committed `eval-run.txt`.

I didn't change the rubric between runs. The prompt the harness sent to the grader for
pkg-14 is byte-for-byte the same in all five runs, so the differences between runs 2–5
are the grader varying, not edits.


**Package analysis**

[Pick one scored package (`pkg-01` through `pkg-20` — the four `calib-` packages are never
scored). Name it by id, say what your rubric decided and what the gold label said, and
explain why your rubric read it that way.]
`pkg-14` (zellij-org/zellij#5174, "OSC color sequences leak into terminal on session
reattach via SSH"). My rubric rejected it in all five runs, and it's my only miss in the
final run. The gold label was accept, with the note "honestly scoped-down: reattach
handshake fix with a regression-window repro; defers the untestable Windows variant and
says so; arguable on the deferral, ready as scoped."

In the final run, `eval-run.txt` lists three failed checks: "failed: diagnosis, scope,
executable". Across the five runs, though, only one of them failed every time:

| Run | Failed checks on pkg-14 |
|---|---|
| 1 | `executable` |
| 2 | `executable` |
| 3 | `executable` |
| 4 | `scope`, `executable` |
| 5 (committed) | `diagnosis`, `scope`, `executable` |

So `executable` is what rejects pkg-14. `scope` and `diagnosis` came and went between runs
with the same files.

**Why `executable` fails it.** The plan names its files as "the client attach/reattach path
in `zellij-server` (session connection handling) and `zellij-client`'s terminal query
issuance; exact functions to be pinned in the PR after tracing the query issuance with debug
logs, which I have working (the leak's origin is visible in `zellij --debug` output)." My
check fails a plan when "key unknowns are left for the build to discover", and "exact
functions to be pinned in the PR" says outright that the build will find the change site.
The gold label reads the same sentence as enough to start from: two named code areas, plus
a tracing method the author already has working. My check has no way to give credit for a
stated method of closing an unknown. It only asks whether the unknown is still open.

**Why `scope` and `diagnosis` sometimes joined it.** Both times, the grader read the second
control in the repro block:

> After `rm -rf ~/.cache/zellij` (thread observation): the next attach is clean, the one
> after leaks again.

The plan's stated cause is that "the reattach path wires the client's stdin to the session
before the query responses have been consumed." The plan explains the control ("with an
empty cache the color data is refetched along the fresh-attach path once"), but that makes
the cache part of the mechanism. Then the plan's not-in-scope line defers "any change to how
theme detection caches its results". `scope` fails a plan when "the repro evidence points at
something the plan lists as out of scope", and `diagnosis` fails one when "any repro step or
control shows the behavior happening where the cause says it cannot". Whether the plan's own
explanation of the control counts as covering it is a judgment call, and the grader made it
differently from run to run. Gold's note concedes the same tension ("arguable on the
deferral").


**Check rationale**

[Quote one check from the `rubric.md` you uploaded to `tools/plan-check/`, exactly as it reads now.
Then say why it reads that way — what you revised to get there, or what you rejected in
favour of it.]
> Passes if the plan is one bounded change: it names the files or code areas it will
> change, says what it will not touch, and every change listed serves the diagnosed cause.
> Leaving part of the issue for later passes if the plan says so explicitly. Fails if the
> plan adds unrelated work ("while I'm here" renames, refactors, docs passes, migrations,
> new flags), or if the repro evidence points at something the plan lists as out of scope.

(`scope`, from `rubric.md`.) The first half is the obvious part: scope creep is a named
failure family, and `scope-creep` came back 4/4, so the "while I'm here" clause does the job
it was written for.

The two clauses that decide the hard cases are the ones I added deliberately, and they pull
against each other. "Leaving part of the issue for later passes if the plan says so
explicitly" is there because a first contribution that honestly does less than the whole
issue is better than one that promises all of it — I did not want the check to punish a plan
for being small. The final clause is there because an out-of-scope line is also the easiest
place to hide the part you could not work out, so I wanted a deferral to be checkable against
something rather than taken on the author's word, and the repro evidence is the only thing in
the package that can do that job. What I rejected was judging the not-in-scope line on its
own terms, which would have made it unfalsifiable — any deferral would pass simply by being
written down.


**Trade-offs**

[Every check gives something up. Any one of these is a complete answer: a package whose
result it changes, a canary you re-ran with `--only`, a case you accept it will miss, or a
stated reason nothing changed elsewhere. "Nothing changed, and here is how I know" earns
the point in full when the reason follows.]
The check that costs me the most is `executable`. What it gives up is the plan that names
its code areas and how it will find the exact site, but not the site itself. It changes the
result for two packages:

- **pkg-14, in every run.** `executable` failed it all five times, and in runs 1–3 it was
  the only check that failed. That makes `executable` the reason pkg-14 is rejected, not
  `scope`. Without it, pkg-14 would have been accepted in at least those three runs.
- **pkg-03, once.** In run 3, with the same files as every other run, the grader rejected
  pkg-03 on `executable` alone, and that run dropped to 18/20. Gold's note on pkg-03 says
  the plan has "tool-by-tool -- support named as a checked unknown". A plan that names its
  unknown honestly is the kind this check sits right on the edge of. Most runs it passes,
  and sometimes it doesn't.

I know nothing else moved between runs because the grader got the same input every time.
The prompt for pkg-14 is byte-for-byte the same in all five runs, and the rubric hash in
`eval-run.txt` (`rubric.md  sha256:9bf2c84e206f8d30`) matches the `rubric.md` uploaded to
`tools/plan-check/`. So the only difference between run 3's 18/20 and the 19/20 runs is the
grader varying on pkg-03. In the final run, every miss is in `clear-accept` (6/7), and
`scope-creep`, `thread-convention`, `unbuildable` and `wrong-cause` are all full marks. The
rubric is strict, not blind. It costs me accepts on plans that were ready, never accepts on
plans that weren't.

I accept that it will miss plans like pkg-14, and I didn't loosen it. Loosening it would
mean passing a plan that defers the exact change site if it names a method for finding it.
For a tool whose real job is deciding whether *my own* plan is ready to post, I'd rather it
hold a plan that says "to be pinned in the PR" than wave one through. 19/20 clears the bar,
and every category floor is met.


---

Related paths: `plan.md` and `eval-run.txt` in this directory; your skill's files in
`tools/plan-check/`.
