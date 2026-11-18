# Adding an endpoint to list an organization's applications in simpler-grants-gov

## Overview

[simpler-grants-gov](https://github.com/HHS/simpler-grants-gov) is HHS's open source rebuild of
Grants.gov. Its Flask API had no way to list the applications that belong to an organization:
[issue #13175](https://github.com/HHS/simpler-grants-gov/issues/13175) asked for a new
`POST /v1/organizations/:organization_id/applications/list` endpoint, and the route, schemas, and
service it needed did not exist yet.

I opened [PR #13257](https://github.com/HHS/simpler-grants-gov/pull/13257), which adds the
endpoint and 11 tests. As of 11/10/2026 the PR is open and awaiting maintainer review.

**What the endpoint does**

- Returns the organization's full, unpaginated list of `IN_PROGRESS` and `SUBMITTED`
  applications as `application_id` and `application_name`, ordered by `created_at` descending.
- Only members of that organization with the required privileges can call it. Access is checked
  with `get_organization_and_verify_access(..., privileges={Privilege.VIEW_APPLICATION})`, which
  also requires `VIEW_ORG_MEMBERSHIP`.
- Documents 200, 401 (no JWT), 403 (missing privilege or wrong org), and 404 (unknown org).

**How it is built**

I followed the existing `roles/list` endpoint (`organization_routes.py`, line 118), which already
had the access check and response shape I needed:

| Layer | File | Change |
|---|---|---|
| Route | `api/src/api/organizations_v1/organization_routes.py` | `organization_list_applications`, JWT auth, no body, no pagination |
| Schemas | `organization_schemas.py`, `api/src/api/schemas/shared_schema.py` | `OrganizationApplicationSchema`, `OrganizationListApplicationsResponseSchema` |
| Service | `api/src/services/organizations_v1/list_organization_applications.py` | Status filter, newest-first order, competitions loaded with `selectinload` |
| Tests | `api/tests/src/api/organizations_v1/test_list_organization_applications.py` | 11 tests |

Two deviations from the issue, both called out in the PR:

- **Name fallback.** The issue says `application_name` must never be null. The name falls back to
  the competition title, but `competition_title` is also nullable, so I added a final
  `"Untitled Application"` fallback.
- **No OpenAPI regeneration.** The daily `ci-openapi.yml` job regenerates the spec, and committing
  it by hand could conflict with that job.

## What happened

### Reproducing a feature that doesn't exist yet

There is no broken behavior to show for a missing feature, so my repro-check tool first failed
its `artifact-backs-claim` check: a plain 404 could just mean an auth or URL problem. I fixed the
evidence by adding a control request to an endpoint that does exist:

```text
$ curl -i -X POST http://localhost:8080/v1/organizations/<any-uuid>/applications/list
HTTP/1.1 404 NOT FOUND

$ curl -i -X POST http://localhost:8080/v1/organizations/<any-uuid>/roles/list
HTTP/1.1 401 UNAUTHORIZED
```

`roles/list` gets far enough to reject the missing token, while `applications/list` is never
routed at all. That shows the route is missing, not that the request was wrong.

I then wrote the new test file before the implementation and ran it next to the existing
`roles/list` tests as a control:

```text
test_list_organization_roles.py ... 5 passed
test_list_organization_applications.py::test_list_organization_applications FAILED
E       assert 404 == 200
request.path=/v1/organizations/4799bf6f-.../applications/list  response.status_code=404
```

### Local setup

The API runs as a Docker Compose stack (Postgres, OpenSearch, S3/SQS/DynamoDB mocks, a mock
OAuth2 server) inside WSL2. Two things got in the way:

- Docker Desktop kept losing its connection to my Ubuntu WSL distro because WSL ran out of
  memory. I fixed it with a `.wslconfig` that caps WSL's memory and storage use.
- My first `make init` failed while pulling the service images
  (from `beat-2-contribution/output.md`):

  ```text
  failed to copy: httpReadSeeker: failed open: ... net/http: TLS handshake timeout
  make: *** [Makefile:187: setup-postgres-db] Error 1
  ```

  Re-running `make init` reused the cached build and finished the migrations, and after
  `make start` the API answered its health check:

  ```text
  $ curl http://localhost:8080/health
  { ..., "message": "Service healthy", "status_code": 200 }
  ```

### A change I made: the privilege setup in my tests

My first version of the test fixture only granted the user `VIEW_ORG_MEMBERSHIP`, so the
happy-path test was rejected by the access check instead of returning 200. Reading
`get_organization_and_verify_access` showed that it requires `VIEW_ORG_MEMBERSHIP` plus every
privilege passed in, so the test user also needed `VIEW_APPLICATION`. I updated the fixture to
grant both, and added a separate 403 test for a user who is missing `VIEW_APPLICATION`. Because
requiring both privileges might be stricter than intended, I also asked about it in the PR.

### A change I made to my tooling: issue-scout's `repo-in-use` rule

Before I found this issue, my issue-scout tool (GitHub search plus a five-check rubric) accepted 4
of 5 candidates on its first scan. Checking those accepts by hand with `gh api` found two misreads:

- `martoniel/nexuspay#34` passed `repo-in-use`, but every commit was from the day the repo was
  created.
- `legend-esc/carbonchain#1190` passed `unclaimed`, but an unlinked merged PR (#1209) had already
  done part of the work.

I tightened the rubric:

| Check | Before | After |
|---|---|---|
| `repo-in-use` | Recent commits exist | Commits on at least 2 days, 7+ days apart, or a maintainer response |
| `unclaimed` | No linked PR or assignee | An overlapping unlinked PR is graded unclear, which counts as a fail |

I also added an unquoted `fastapi OR flask` query and capped the top 5 at 2 per query and 1 per
repo so one noisy query couldn't fill the list. Even then, the broad keywords pulled in frontend
issues that only mentioned pagination, so I added `api` and `backend` terms, which surfaced
#13175.

### Result

The same test command that failed with `assert 404 == 200` now passes all 11 tests:

```text
make test args="-x -vv tests/src/api/organizations_v1/test_list_organization_applications.py"
```

They cover the 200 response, an empty org, both name fallbacks, org isolation, the status filter,
newest-first order, 403 (missing privilege or wrong org), 404, and 401. I also ran the whole
`tests/src/api/organizations_v1/` folder (including the `roles/list` tests) and `make format`,
`make lint`, and `make lint-security` before opening the PR. The PR is open and waiting on
maintainer feedback.

## Limitations and what's next

- **Open design questions.** I asked reviewers whether `ACCEPTED` applications should be included,
  whether `"Untitled Application"` is an acceptable label, and whether requiring both
  `VIEW_ORG_MEMBERSHIP` and `VIEW_APPLICATION` is the intended access rule. Any of these could
  change the filter or the access check.
- **Unpaginated by design.** The issue asks for the full list. That is fine for typical
  organizations but could get slow for one with many applications; pagination would be a
  follow-up if that happens.
- **Deprecation warning in the tests.** One library call in the new test file doesn't fail but
  emits a deprecation warning. Next time I would switch to the non-deprecated call before opening
  the PR.
- **Tooling fit.** repro-check and my voice-guard were written with bug reports in mind, so they
  were awkward for a feature request. Next time I would build them to handle features from the
  start, and spend more time reading the repo before planning.
