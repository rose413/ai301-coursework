# Evidence guide: where evidence lives in a PR package

Every practice submission (`eval/packages/*.md`) has the same
sections: `## Repo facts`, `## Issue`, `## Thread highlights`,
`## Plan context`, and `## Candidate PR`, which holds `### Title`,
`### Description`, `### Commits`, `### Diff`, and `### Test evidence`.
In live mode the same evidence comes from your own files and the
scoped repo:

- `plan.md`: your posted plan, including its Deviations section (the
  house plan and house repro pack for a house-chain student)
- your branch's diff: `git diff main...HEAD` from your working copy
  (`--stat` for the file list), and `git log --oneline main..HEAD` for
  the commits
- `test_evidence.md` (step 8): your captured before/after output and
  the repo's checks
- `pr_draft.md` (step 9): your draft PR title and description
- the issue thread (`gh issue view <number> --comments`), the repo's
  PR template, and `docs/CONTRIBUTING.md`

## Plan fidelity (harness category: silent-drift)

**Checks:** diff-matches-plan, description-matches-diff.

**Where it lives in a practice submission:**
- The plan: `## Plan context`. The `Files:` line names what the plan
  will change, the not-in-scope line (usually `Not in scope:`, but any
  sentence that rules work out counts) names what it rules out, the
  `Plan:` text lists the steps it promises, and any deviation note or
  deferral is written into this block.
- What changed: `### Diff` (each `--- a/` / `+++ b/` pair is one file;
  each `@@` is one hunk) and `### Commits`.
- What the PR claims: `### Title` and `### Description`, including any
  checklist inside the description.

**Where it lives in your own PR:**
- The plan: `plan.md`, its files / scope line, its not-in-scope line,
  its steps, and its Deviations section.
- What changed: `git diff --stat main...HEAD` for the file list and
  `git diff main...HEAD` for the hunks.
- What the PR claims: `pr_draft.md`, the title and every sentence and
  checklist item of the description.

**What good looks like:**
- Every changed file is named by the plan (the file or its folder; "a
  changelog entry" names the changelog file), or is a test, fixture,
  or changelog file for the planned change, or is covered by a
  deviation note. Every step the plan promises has a hunk that
  delivers it, or a deviation note saying why not.
- Every claim in the description is true of the diff in both
  directions: what it says was added is in the diff, and what it says
  was not changed or added ("no options added", "exactly the plan")
  is not. A deviation or deferral recorded in the plan is restated in
  the description.

**Silent drift looks like:**
- *More than the plan:* a file the plan never names (calib-03's
  `print/property.js`), or a new option, flag, config key, or rename
  the plan does not ask for or rules out, with no note.
- *Less than the plan:* a promised step with no hunk and no note, or a
  description claiming something the diff does not contain (tests for
  cases with no fixture in the diff, "docs now document it" with no
  docs change).
- A fidelity claim ("implements the plan exactly") over a diff that
  contradicts it is drift, not a comms problem.

## Test evidence (harness category: not-tested)

**Checks:** repro-re-run, repo-checks-run.

**Where it lives in a practice submission:**
- The proof: `### Test evidence`, its commands, their output lines,
  and any `# before:` / `# after:` lines, plus prose lines about the
  suite.
- What it must prove: the `Test plan:` text in `## Plan context`
  (every repro, failure mode, and expected-after it names), and the
  repro evidence quoted in the same block (the before result).
- What the repo asks to be run: `## Repo facts` (for example, "tests
  added and passed", "all code checks passed").

**Where it lives in your own PR:**
- The proof: `test_evidence.md`, your captured commands and output,
  before and after, and the repo's checks.
- What it must prove: the test plan in `plan.md`, and your repro from
  Unit 2 (or the house repro pack).
- What the repo asks to be run: the PR template and
  `docs/CONTRIBUTING.md`.

**What good looks like:**
- Every repro and failure mode the test plan names is re-run on the
  branch, each shown with its command and a concrete after result: the
  printed output, error text, or exit code, as output lines or as a
  `# after:` line that states the result itself (calib-01's `# after:`
  followed by `parsed OK`). The after result is the plan's
  expected-after and differs from the before.
- Each suite or check the plan or repo asks for is reported with the
  command or suite named and its outcome (for example, "`cargo test -p
  ignore` passes (214 tests)").

**Not tested looks like:**
- A judgment instead of a result: "works now", "tested locally",
  "verified on my machine", "matches the expected output byte for
  byte" with no output shown.
- Only a control or the unchanged path re-run, never the failing case.
- One failure mode re-run when the test plan names two (calib-04).
- "Tests pass" or "CI is green" with no command or suite named, or a
  requested suite never mentioned.

## Diff quality (harness category: unreviewable)

**Checks:** clean-diff, commit-history (preferred).

**Where it lives in a practice submission:**
- `### Diff`: every `+` and `-` line in every hunk.
- `### Commits`: each commit's hash and message.

**Where it lives in your own PR:**
- `git diff main...HEAD`: every `+` and `-` line in every hunk.
- `git log --oneline main..HEAD`: each commit's message.

**What good looks like:**
- Every added or removed line serves the planned change, its tests,
  its changelog entry, or a noted deviation. The fix can be read on
  its own, with nothing else riding along.
- Each commit message says what that commit changes.

**Unreviewable looks like (the debris tells):**
- A debug print or log line (`eprintln!("DEBUG ...")`, `print(...)`,
  `console.log`).
- Commented-out code, including an earlier attempt left in.
- A dead function or unused variable, especially behind
  `allow(dead_code)` or a leading underscore.
- An added TODO or FIXME.
- Formatting, whitespace, import-order, or re-indent churn on lines
  the fix does not otherwise need, or lines removed and re-added
  identical.
- A hunk unrelated to the change.
- Commit messages like "wip", "fix", "fmt", or "misc cleanups while
  debugging" (preferred check only).

## Standards and comms (harness category: standards-wall)

**Checks:** template-asks-met, ai-disclosure, maintainer-direction,
title-says-what (preferred).

**Where it lives in a practice submission:**
- What the repo asks: `## Repo facts`. The `pull requests:` line
  holds the template's sections and checklist, the issue-link form,
  and required files (changelog, whatsnew, release notes). The
  `contribution policy:` line holds the contributing rules and any AI
  policy, including what a disclosure must state.
- Maintainer direction: `## Thread highlights`, comments marked
  OWNER, MEMBER, or COLLABORATOR.
- Where the PR honors them: `### Title`, `### Description`
  (sections, checklist, issue link, disclosure), and `### Diff`
  (required files present).

**Where it lives in your own PR:**
- What the repo asks: the repo's PR template (every section of the
  Path Review template) and `docs/CONTRIBUTING.md`.
- Maintainer direction: your issue's thread on the scoped repo.
- Where the PR honors them: `pr_draft.md` (every template section,
  the `Closes #N` line, and your AI disclosure) and `git diff
  main...HEAD` (required files present).

**What good looks like:**
- Every stated ask is met with real content: each template section or
  checklist item filled (or marked N/A with a reason), the issue link
  in the form asked ("Closes #N", "fixes #N"), and every file the repo
  asks for present in the diff.
- When the repo's policy or template asks for AI disclosure, the
  description discloses it in your own words with everything the
  policy asks for (for example, the tool used and the extent of the
  help). Your own PR always discloses: course work is AI-assisted by
  design.
- Each explicit maintainer direction in the thread is followed, or
  addressed in the description.
- A repo with no template and no stated policy asks for nothing more.

**A standards wall looks like:**
- A required checklist left out, the issue-link line missing, or a
  required changelog / whatsnew entry absent from the diff (the PR is
  good and the template is still ignored).
- No AI disclosure where the repo's policy requires one, or a
  boilerplate line that omits what the policy asks for.
- A PR that goes against a maintainer's stated direction without
  saying why.
- Whether the description's claims match the diff is plan fidelity,
  above, not standards.
