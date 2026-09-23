# Rubric: is this a good first issue?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| maintainer-active | Repo facts: last 5 default-branch commits; maintainer first-response sample; Comments section for maintainer author_association | Pass if at least 1 of the last 5 default-branch commits is within 90 days of the capture date OR the maintainer first-response sample shows at least 1 owner/member/collaborator response within 90 days; otherwise fail. | required |
| repo-in-use | Repo facts: archived status and last push to any branch | Pass if the repository is not archived AND there has been a push to any branch within 180 days of the capture date; otherwise fail. | required |
| newcomer-scope | Issue body and Comments section; use the scope guidance in references/evidence-guide.md | Pass if the issue states a concrete, actionable contribution with a reasonably clear desired result or acceptance criteria, and is not an umbrella/tracking issue, pure usage/support question, unresolved design discussion, or work explicitly described by a maintainer as requiring major/core-internal architectural changes. Multiple possible causes or suggested implementation approaches do not by themselves fail the check when the issue still identifies a concrete problem and desired outcome. A multi-file documentation task passes when its requested changes are explicitly enumerated. | required |
| unclaimed | Repo facts: this issue's assignees and linked PRs; Comments section for claim comments and mentioned PRs | Pass if there is no current assignee, no open linked PR, no unresolved claim/working-on-it comment, and no linked PR or claim activity within 365 days before the capture date that indicates a contributor was actively attempting the issue. Older closed PRs and old claims alone do not fail this check when there is no activity within that 365-day window. | required |
| contribution-policy | Repo facts: contribution policy and dedicated AI policy files | Pass if there is no outright ban on AI-assisted or AI-generated contributions. Conditions such as disclosure, human review, testing, or understanding the submitted work do not count as bans. | required |

## Verdict rule

Accept if every required check passes. Preferred checks, if added later, never change the verdict and only help rank accepted issues. Unclear counts as fail.

