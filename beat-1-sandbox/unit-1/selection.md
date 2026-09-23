# Unit 1 — Issue Selection

Path: `beat-1-sandbox/unit-1/selection.md`

Record of the issue carried into Unit 2, and of the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in
the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the
wrong label is not graded.

---

## Selected issue

**Issue link**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/38

**Verdict output**

[Your skill's live-mode output for this issue, pasted verbatim and ending with the
fenced JSON verdict block. A summary does not satisfy this field.]

**The verdict must record `accept` for this issue.** Choose an issue your own skill
accepts. If your skill rejects every candidate you try, that is a signal about your
rubric rather than about the issues: revise it and re-run — retries are unlimited and a
partial re-run costs about $0.20 — or run the skill on different candidates. Output
recording `reject` for the issue you chose earns no credit for this field.

```
All five required checks came back clean on all three, so the ranking is what does the work here.

Repo-level evidence (shared; measured against today, 2026-09-23):
- Newest main commit 2026-09-16 by Aburke225 (COLLABORATOR) — 7 days old
- archived: false, pushed_at: 2026-09-16 — 7 days old
- Policy lives at docs/CONTRIBUTING.md (one click from the README, not the root); it covers branches, commits, CI, and seeded-bug xfail markers, and says nothing about AI-assisted work. Silence passes.
- The repo has zero PRs in any state, so no issue can carry a linked or informally-mentioned PR.

Ranked read-out — all three accepted:

1. #38 — Add integration tests for authentication edge cases (best fit)
Fit: 3–5h, the smallest of the three, and the only one whose acceptance criteria are enumerated exhaustively — four named cases (expired token, malformed token, no Authorization header, token signed with a different secret). Python/FastAPI against an existing api/middleware/auth.py; testable end-to-end inside course time, which is exactly the "focused change I can understand and test" the profile asks for.
- maintainer-active pass · repo-in-use pass · newcomer-scope pass · unclaimed pass · contribution-policy pass

2. #44 — Test coverage for core/services/review_service.py below 40%
Fit: 5–7h, one Python test file, still bounded. Ranked under #38 because "success, partial failure, and full failure cases" is a looser target than four named cases, and review_service is described as the most critical service — more code to understand before the first line of test.
- All five required checks pass

3. #35 — Implement a webhook system for review-ready notifications
Fit: weakest of the three. 8–12h, tier-3, two new files (api/routes/webhooks.py, core/services/webhook_service.py) — a new subsystem rather than a focused change, and the closest of the three to the "large architectural change" the profile wants to avoid.
- All five required checks pass

One tension worth naming: #35 passes newcomer-scope on the rubric's stated condition — it's not an umbrella issue, not a support question, has no design debate in the thread (zero comments), and no maintainer statement that it touches core internals. It states a concrete desired result and names its files. But the rubric has no effort or size gate, so an 8–12h tier-3 greenfield subsystem clears the same bar as a 3–5h test addition. The rubric decided this one, not me. If you want that excluded, that's a rubric edit — a required check on estimated effort or the tier-* label — not a judgment call at grading time.

[
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s3/issues/38",
    "checks": [
      {"name": "maintainer-active", "grade": "pass", "evidence": "Newest main commit 2026-09-16 by Aburke225 (COLLABORATOR), 7 days before capture date 2026-09-23 — within 90 days."},
      {"name": "repo-in-use", "grade": "pass", "evidence": "Repo archived: false; pushed_at 2026-09-16, 7 days before capture date — within 180 days."},
      {"name": "newcomer-scope", "grade": "pass", "evidence": "Body enumerates four concrete test cases (expired token, malformed token, no Authorization header, token signed with a different secret) and names the files; not umbrella, support, design debate, or maintainer-flagged core work."},
      {"name": "unclaimed", "grade": "pass", "evidence": "assignees=[]; 0 comments; timeline shows only 'labeled' events; repo has 0 PRs in any state."},
      {"name": "contribution-policy", "grade": "pass", "evidence": "docs/CONTRIBUTING.md and .github/PULL_REQUEST_TEMPLATE.md contain no mention of AI-assisted or AI-generated contributions; no AI_POLICY file exists."}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s3/issues/44",
    "checks": [
      {"name": "maintainer-active", "grade": "pass", "evidence": "Newest main commit 2026-09-16 by Aburke225 (COLLABORATOR), 7 days before capture date 2026-09-23 — within 90 days."},
      {"name": "repo-in-use", "grade": "pass", "evidence": "Repo archived: false; pushed_at 2026-09-16, 7 days before capture date — within 180 days."},
      {"name": "newcomer-scope", "grade": "pass", "evidence": "Concrete ask with a named target file and stated paths to cover (success, partial failure, full failure); not umbrella, support, design debate, or maintainer-flagged core work."},
      {"name": "unclaimed", "grade": "pass", "evidence": "assignees=[]; 0 comments; timeline shows only 'labeled' events; repo has 0 PRs in any state."},
      {"name": "contribution-policy", "grade": "pass", "evidence": "docs/CONTRIBUTING.md and .github/PULL_REQUEST_TEMPLATE.md contain no mention of AI-assisted or AI-generated contributions; no AI_POLICY file exists."}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s3/issues/35",
    "checks": [
      {"name": "maintainer-active", "grade": "pass", "evidence": "Newest main commit 2026-09-16 by Aburke225 (COLLABORATOR), 7 days before capture date 2026-09-23 — within 90 days."},
      {"name": "repo-in-use", "grade": "pass", "evidence": "Repo archived: false; pushed_at 2026-09-16, 7 days before capture date — within 180 days."},
      {"name": "newcomer-scope", "grade": "pass", "evidence": "States a concrete desired result (register a callback URL, POST the review payload on completion) and names both new files; no umbrella list, no support question, no design debate in the thread (0 comments), and no maintainer statement that it requires core-internal changes — passes as written despite 8-12h tier-3 size, which the rubric does not gate."},
      {"name": "unclaimed", "grade": "pass", "evidence": "assignees=[]; 0 comments; timeline shows only 'labeled' events; repo has 0 PRs in any state."},
      {"name": "contribution-policy", "grade": "pass", "evidence": "docs/CONTRIBUTING.md and .github/PULL_REQUEST_TEMPLATE.md contain no mention of AI-assisted or AI-generated contributions; no AI_POLICY file exists."}
    ],
    "verdict": "accept"
  }
]
```

---

## Eval iterations

Quote source text directly in each field below. Paraphrase does not satisfy them.

**Run history**

[The agreement score of each run you did, in order. A single run is a complete answer if
only one run occurred. **The last score in your list must match the agreement line in the
`eval-run.txt` you committed** — that file is the record of your final run.]
Single run: 19/20 agreement (claimed 4/4, clear-accept 8/8, dead-repo 3/3, policy 1/1,
scope 3/4). This is the only full run I did, and it matches the agreement line in the
committed `eval-run.txt`.


**Issue analysis**

[One scored issue, identified by id (`issue-01` through `issue-20`; the `calib-`
issues are not scored). State your rubric's decision, the gold label, and the
reasoning that produced your rubric's result.]
`issue-20` (excalidraw/excalidraw#11811, "Add company logo shape to the toolbar"). My
rubric's verdict was accept; the gold label was reject. All five required checks passed
on the literal evidence: the repo had commits within the last day (`maintainer-active`),
was not archived and recently pushed to (`repo-in-use`), had no assignees or linked PRs
(`unclaimed`), had no stated AI-contribution ban (`contribution-policy`), and the issue
body lists concrete acceptance criteria ("Success looks like: logo tool in the shapes
toolbar → place/resize/move like other elements → correct export"), which satisfied
`newcomer-scope`.

What my rubric missed: the issue was filed by `cursor[bot]` (author association `NONE`)
and had zero comments, no labels, and no visible maintainer triage — nothing indicates a
maintainer actually wants this feature. The issue also admits real unresolved scope
("Logo asset TBD", "possibly app wiring in `excalidraw-app` if needed"). My
`newcomer-scope` check only tests whether the issue itself states a clear contribution
and acceptance criteria; it doesn't test whether a maintainer has endorsed or triaged the
request. On a large, high-traffic repo, an untriaged bot-filed feature request can look
well-specified and still never be something the maintainers intend to accept, which is
likely why gold reads this as reject.


**Check rationale**

[One check from the `rubric.md` uploaded to `tools/issue-select/`, quoted as it is
currently written, with the reasoning behind its current form.]
> Pass if the issue states a concrete, actionable contribution with a reasonably clear
> desired result or acceptance criteria, and is not an umbrella/tracking issue, pure
> usage/support question, unresolved design discussion, or work explicitly described by
> a maintainer as requiring major/core-internal architectural changes.

(`newcomer-scope`, from `rubric.md`). I wrote this check to filter out the issues that
waste a newcomer's time even when they look inviting — vague discussion threads,
"tracking" issues with no single deliverable, or things a maintainer has already said
need a rewrite. Requiring a stated desired result was meant to catch issues that are
really open design questions in disguise.


**Trade-offs**

[What the quoted check gives up. Any one of these is a complete answer: an issue whose
result it changes, a canary you re-ran with `--only`, a case you accept it will miss, or a
stated reason nothing changed elsewhere. "Nothing changed, and here is how I know" earns
the point in full when the reason follows.]
This check only reads the issue's own text for clarity of scope — it does not check
whether a maintainer has actually engaged with or endorsed the issue, and it has no
effort or size gate. That's exactly what let `issue-20` through, and what let #35 (the
8–12h greenfield webhook subsystem) pass alongside #38 in live mode despite being much
closer to a "large architectural change" than a first issue should be. I could tighten
the check to require a maintainer comment/label or an estimated-effort ceiling, but I
chose not to, because plenty of good, legitimate first issues — including small,
self-contained ones on quieter repos — sit uncommented or lack an explicit size label
without being bad candidates. I'm accepting that my rubric will pass through some
untriaged or larger-than-ideal issues, and that picking the right one from an accepted
set is left to fit-ranking rather than the verdict itself.


---

## Selection rationale

Graded on whether all three are answered, in your own words. Not on how good the
reasoning is, and not on length — a short honest answer to each earns the full marks.
This is also the basis for the claim comment you write in Unit 2.

**Selection rationale**

1. The issue's fit to your interests and to the time available.
Issue #38 is scoped at 3–5 hours, the smallest of the three accepted candidates, and
matches my Python/FastAPI background directly. It's a focused, testable change I can
complete and understand fully within the course timeline, rather than something that
risks running long or touching unfamiliar territory.

2. What the verdict identified correctly, and what you weighed that the rubric could not.
All three issues passed every required check identically, so the rubric alone couldn't
distinguish between them — it has no size or effort gate, which is exactly what let #35
(an 8–12h greenfield webhook subsystem) pass despite being much closer to the "large
architectural change" I want to avoid. What I weighed beyond the rubric was exactly that:
#38's four exhaustively enumerated test cases give me the clearest, most bounded target
of the three, and the smallest surface area to misunderstand or get wrong.


3. The anticipated difficulty in claiming it.
Since the repo has zero PRs and zero comments on any of these issues, I don't expect
claim conflicts. The main difficulty I anticipate is getting familiar enough with the
existing authentication flow and test setup to write meaningful assertions for all four
edge cases correctly, rather than the scope of the issue itself.


---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/issue-select/`.
