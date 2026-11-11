# Contribution: Add POST /organizations/:organization_id/applications/list endpoint

Path: `beat-2-contribution/contribution-story.md`

**Contribution Number:** 1
**Student:** Irene Chang
**GitHub Username:** rose413
**Issue:** [HHS/simpler-grants-gov#13175](https://github.com/HHS/simpler-grants-gov/issues/13175)
**Pull Request:** [HHS/simpler-grants-gov#13257](https://github.com/HHS/simpler-grants-gov/pull/13257)

---

## Toolkit

Every story section below ends with a Toolkit block. This section keeps one dated entry per
week of tool work.

**Unit 5: Built issue-scout and tightened its rubric after checking its accepts by hand**

I built issue-scout: `gh search issues` queries based on my fit profile, free ranking filters
(drop assigned issues, drop anything not updated in 90 days, sort by fewest comments), and a
five-check rubric that grades the top 5. Scan 1 accepted 4 of 5, but checking those accepts
against the live repos with `gh api` turned up two misreads. martoniel/nexuspay#34 passed
`repo-in-use` even though every commit was from the day the repo was created, and
legend-esc/carbonchain#1190 passed `unclaimed` even though an unlinked merged PR (#1209) had
already done part of the work. So I tightened `repo-in-use` (commits on at least 2 days, 7+ days
apart, or a maintainer response), graded overlapping unlinked PRs as unclear (counts as fail),
added an unquoted `fastapi OR flask` query, and capped the top 5 at 2 per query and 1 per repo.

**Unit 6: Voice-guard and repo-memory file for the claim comment**

The voice-guard was too strict on my claim comment's wording because its rules assumed a
bug-fix issue, and mine asks for a new feature (a missing endpoint). I made the rules more
concise and changed them so they also fit feature requests.
TODO: name the exact rule you changed.

**Unit 7: Ship-gate could not read `gh pr create --help` from PowerShell**

At first, the ship-gate could not read `gh pr create --help` on my real branch, because Claude
could not run it from my PowerShell setup. I changed the ship-gate to read the help through the
PowerShell terminal, and after that it could check my PR command.

**Unit 8: PR-monitor and playbook**

- Response rules: my replies to a reviewer open by thanking them and answering their question
  directly. They never argue or push for a merge.
- Staleness thresholds: TODO: the usual wait from five recent merged PRs, plus a few days.
- Nudge etiquette: one polite nudge on the PR thread after the threshold passes, tagging only
  the maintainer who triaged the issue.
- Walk-away criteria: I close my PR if a maintainer says the work isn't wanted or another PR
  replaces it. My close note thanks them and links the replacement.

---

## Why I Chose This Issue

I searched for issues about APIs and backend development, because I want to get better at API
development and learn good practices. The scout's first results mixed in frontend issues that
only mentioned API topics like pagination. So I added keywords like `api` and `backend` to get
real backend issues. I picked #13175 because it is a self-contained backend task (one new
endpoint) in an active, well-tested Flask codebase, which fits my profile.

The scout fell short on the wild search because its keywords were too broad. Next time I would
make the query wording specific to the kind of development I want to do.

Claim comment: https://github.com/HHS/simpler-grants-gov/issues/13175#issuecomment-6724732317

**Toolkit**

- Tool: issue-scout. I ran `gh search issues` queries from my fit profile and graded the top
  candidates with my rubric.
- What it produced: a ranked list of candidate issues. I chose #13175 from it after adding
  backend keywords.
- Difficulties: the first queries returned mostly frontend issues with a few API terms mixed in.
- What changed: I added `api` and `backend` keywords. I would make the query wording more
  specific to backend development.

---

## Understanding the Issue

**Problem Description**

The API has no `POST /v1/organizations/:organization_id/applications/list` endpoint. The route,
schemas, and service method it needs don't exist yet.

**Expected Behavior**

The endpoint returns the full, unpaginated list of the organization's open and submitted
applications (`application_id` and `application_name`), newest first. Only members of that
organization with the VIEW_APPLICATION privilege can call it.

**Current Behavior**

The route doesn't exist, so a request returns `404 NOT FOUND`.

**Affected Components**

The `api/` folder: the organizations_v1 routes and schemas, a new service in
`api/src/services/organizations_v1/`, and a new test file in
`api/tests/src/api/organizations_v1/`.

**Toolkit**

- Tool: repro-check. I graded my claim comment and repro report against the issue.
- What it produced: an accept verdict. All six required checks and the control run passed, and
  it suggested adding a timeframe for my next update and dropping "Draft" from the title.
- Difficulties: `artifact-backs-claim` was hard to pass because this is a new feature, not a
  bug, so there was no broken behavior to show.
- What changed: I added curl output showing `applications/list` returns 404 while the existing
  `roles/list` returns 401, which proves the route is missing.

---

## Reproduction Process

**Environment Setup**

- Windows 11 Home (10.0.26200), WSL2 with Ubuntu 26.04, Docker Desktop 4.47.0 (Engine 28.4.0)
- simpler-grants-gov `main` at `d16806e`, Python 3.14 in the API container
- Setup: clone the repo, then from `api/` run `make init` and `make start`
  (per [DEVELOPMENT.md](https://github.com/HHS/simpler-grants-gov/blob/main/DEVELOPMENT.md))

Challenge: Docker Desktop kept failing to connect to my Ubuntu WSL distro because WSL ran out
of memory. I fixed it by adding a `.wslconfig` file that limits WSL's memory and storage use.

**Steps to Reproduce**

1. From `api/`, run `make init` and `make start`.
2. Run `curl -i -X POST http://localhost:8080/v1/organizations/<any-uuid>/applications/list`.
   It returns `404 NOT FOUND`. The same request to `.../roles/list` returns `401`, so the route
   is missing; the 404 is not an auth problem.
3. Run the existing roles test as a control, then the new applications test:
   `make test args="-x -vv tests/src/api/organizations_v1/test_list_organization_applications.py"`

**Reproduction Evidence**

- **Commit showing reproduction:** TODO: link the commit in your fork that adds the failing test
- **Screenshots/logs:** full output saved in `beat-2-contribution/output.md`. Key lines:

```text
test_list_organization_roles.py ... 5 passed
test_list_organization_applications.py::test_list_organization_applications FAILED
E       assert 404 == 200
request.path=/v1/organizations/4799bf6f-.../applications/list  response.status_code=404
```

- **My findings:** the existing `roles/list` tests all pass, while the same kind of test against
  `applications/list` fails with 404. That confirms the endpoint is missing, and that
  `roles/list` is a good pattern to copy.

**Toolkit**

- Tool: repro-check. I graded the repro report before posting it.
- What it produced: an accept verdict (all required checks and the control run passed). It
  flagged that the report should say when I'd post my next update.
- Difficulties: describing "current vs. expected behavior" was awkward for a feature request.
- What changed: I framed the current behavior as "roles/list tests pass, applications/list
  returns 404" and added a timeframe for my next update.

---

## Solution Approach

**Analysis**

`POST /organizations/:organization_id/applications/list` was never built. There is no route, no
response schema, no service method, and no tests. The closest existing endpoint is `roles/list`
(`organization_routes.py`, line 118), which already has the access check and response shape
this endpoint needs.

**Proposed Solution**

Follow the `roles/list` pattern: add a route and response schema, add a new service that
queries the org's `IN_PROGRESS` and `SUBMITTED` applications (newest first, name falls back to
the competition title), and add a test file. I am not changing the frontend, anything outside
`api/`, or the generated OpenAPI spec (a daily CI job regenerates it).

**Implementation Plan**

1. Add the route to `api/src/api/organizations_v1/organization_routes.py` and the schemas to
   `organization_schemas.py` and `api/src/api/schemas/shared_schema.py`.
2. Add `api/src/services/organizations_v1/list_organization_applications.py`.
3. Add `api/tests/src/api/organizations_v1/test_list_organization_applications.py`.

Deviations:
- Added an `"Untitled Application"` final fallback, because `competition_title` can also be null
  and the issue says the name must never be null.
- Skipped regenerating the OpenAPI spec, because the daily `ci-openapi.yml` job does it and
  committing it could conflict with that job.

**Toolkit**

- Tool: plan-check. I graded my plan and plan comment against the issue and repro.
- What it produced: an accept verdict on all five checks, plus small fixes: make the OpenAPI
  step optional everywhere, and consider including `ACCEPTED` applications.
- Difficulties: the `executable` and `test` checks failed at first because my plan and test
  list were too vague.
- What changed: I named every file path, schema, and query rule, and listed each test case with
  its expected result.

---

## Testing Strategy

- **Before:** the new test file fails with `assert 404 == 200`, because the route doesn't exist.
- **After:** the same command passes all 11 tests:
  `make test args="-x -vv tests/src/api/organizations_v1/test_list_organization_applications.py"`.
  The tests cover the 200 response, an empty org, both name fallbacks, org isolation, the status
  filter, newest-first order, 403 (missing privilege or wrong org), 404, and 401.
- **Regression:** `make test args="tests/src/api/organizations_v1/"` (including the
  `roles/list` tests), plus `make format`, `make lint`, and `make lint-security`.

**Toolkit**

- Tool: repro-check. I re-ran my reproduction steps against the built change.
- What it produced: the 404 became a 200 with the expected fields, and the test file went from
  failing to 11 passing.
- Difficulties: none met
- What changed: nothing

---

## Implementation Notes

**[11/10/2026]**

Added the route and schemas for `POST /organizations/:organization_id/applications/list`,
modeled on `roles/list`.

**[11/10/2026]**

Added `services/organizations_v1/list_organization_applications.py`. It checks access with
`get_organization_and_verify_access(..., privileges={Privilege.VIEW_APPLICATION})`, filters to
`IN_PROGRESS` and `SUBMITTED`, orders by `created_at` descending, and loads competitions with
`selectinload`. I added the `"Untitled Application"` fallback when I found that
`competition_title` is also nullable.

**[11/10/2026]**

Added `api/tests/src/api/organizations_v1/test_list_organization_applications.py` with 11 tests.
My first test only granted `VIEW_ORG_MEMBERSHIP`. I fixed it once I saw that the access check
requires both privileges.

**Toolkit**

- Tool: Claude (Claude Code), used to find the relevant files and draft code and tests
- What it produced: draft route, service, schema, and test code following the `roles/list`
  pattern
- Difficulties: I had to read each suggestion carefully to make sure I understood it and that it
  matched the issue (for example, the privilege check and the null-name fallback).
- What changed: I reviewed and adjusted the drafts and ran the tests locally before committing.

---

## Pull Request

**PR Link:** https://github.com/HHS/simpler-grants-gov/pull/13257

**PR Description:**

Fixes #13175. Adds `POST /v1/organizations/:organization_id/applications/list`, which returns
the organization's full, unpaginated list of applications (`application_id`,
`application_name`). It follows the same route → service → schema → test structure as
`roles/list`.

- **Route:** `organization_list_applications` in `organization_routes.py`. It uses JWT auth,
  takes no body and no pagination, and documents 200/401/403/404.
- **Service:** new `list_organization_applications.py`. It returns the org's `IN_PROGRESS` and
  `SUBMITTED` applications, newest first. The name falls back to the competition title, then to
  `"Untitled Application"`.
- **Schemas:** `OrganizationApplicationSchema` and `OrganizationListApplicationsResponseSchema`.
- **Tests:** 11 tests covering every acceptance criterion.
- **Questions for reviewers:** Should `ACCEPTED` be included? Is the "Untitled Application"
  label OK? Is requiring both `VIEW_ORG_MEMBERSHIP` and `VIEW_APPLICATION` the intended access
  rule?
- **AI assistance:** I used Claude to understand the codebase and draft code and tests. I
  reviewed and understood everything before submitting.

**Maintainer Feedback:**

- 11/10/2026: none yet

**Status:** Awaiting review

**Toolkit**

- Tool: pr-precheck. I graded my branch, PR title, and description against my plan and the
  issue before opening the PR.
- What it produced: a ready-to-submit verdict and a description that follows the repo's PR
  template. TODO: confirm the verdict.
- Difficulties: TODO: note anything it flagged.
- What changed: I wrote the AI assistance section in my own words.

---

## Learnings and Reflections

**Technical Skills Gained**

- Running a full Docker Compose dev stack inside WSL2, and tuning WSL with `.wslconfig`
- How Flask routes, marshmallow schemas, and service functions connect in a layered API
- SQLAlchemy filtering, ordering, and `selectinload`, plus privilege-based access checks
- Writing pytest tests with factories, and using Claude Code with custom skills to check my work

**Challenges Overcome**

- Docker Desktop wouldn't connect to my WSL2 Ubuntu because of memory limits. I researched how
  Docker and WSL interact and fixed it with a `.wslconfig` limit.
- plan-check failed my `executable` and `test` checks for being vague. I learned to name exact
  files and test cases.
- repro-check is built for bugs, so I had to learn how to show evidence for a missing feature
  (a 404 next to a 401 control).

**What I'd Do Differently Next Time**

I would spend more time understanding the repo before planning, and put more time into making
my tools thorough from the start, so they fit feature requests as well as bugs.

---

## Resources Used

- [simpler-grants-gov DEVELOPMENT.md](https://github.com/HHS/simpler-grants-gov/blob/main/DEVELOPMENT.md): local setup
- The existing `roles/list` route and `test_list_organization_roles.py`: the pattern I followed
- [Issue #13175](https://github.com/HHS/simpler-grants-gov/issues/13175): acceptance criteria
- Microsoft's WSL `.wslconfig` documentation: fixing the Docker/WSL memory problem
