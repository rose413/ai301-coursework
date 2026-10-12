# Rubric: is this pull request ready to submit?

Each check reads the PR package itself: the candidate PR (title,
description, commits, diff, test evidence) against the plan context
(the plan's Files / scope line, its not-in-scope line, its test plan,
and any deviation note), the issue and thread, and the repo facts
(PR template asks and contribution policy). In live mode the same
parts come from `plan.md` and its Deviations section, `git diff
main...HEAD`, the draft title and description, the captured test
output, the issue thread, the repo's PR template, and
`docs/CONTRIBUTING.md` (see `references/evidence-guide.md`).

Checks 1 and 2 catch silent drift, checks 3 and 4 catch not tested,
check 5 catches unreviewable, and checks 6 to 8 catch the standards
wall. A package that passes every required check is a clear accept.

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| diff-matches-plan | Every file and hunk in the diff, read against the plan's Files / scope line, its not-in-scope line, and any deviation note in the plan; then every step the plan promises, read against the diff. | Passes if every changed file is named by the plan (the file or its folder; "a changelog entry" names the changelog file), or is a test, fixture, or changelog / release-note file for the planned change, or is covered by a deviation note, AND every hunk in those files serves the planned change, AND every step the plan promises appears in the diff or its absence is recorded in a deviation note. Fails if the diff changes a file the plan does not name and no note covers it (for example, a second source file rewritten), adds a new option, flag, config key, or rename the plan does not ask for or rules out, or leaves out a planned step with no note. | required |
| description-matches-diff | Every claim in the PR title, description, and checklist (fidelity claims such as "implements the plan exactly", "no changes beyond it"; "tests added for X"; "docs updated"; "changelog added"; "no options added"), each read against the diff. Also any deviation or deferral in the plan, read against the description. | Passes if every claim is true of the diff: each thing the description says was added appears in the diff, and each thing it says was not changed or added does not appear in it. A claim that nothing was added passes if nothing in the diff adds it. If the plan records a deviation or deferral, the description must restate it. Fails if any claim is contradicted by the diff in either direction: it claims something the diff does not contain (a test, a docs update, a fixture that is not there), or denies something the diff does contain (claims "exactly the plan" over a hunk the plan never names), or hides a deviation the plan records. | required |
| repro-re-run | The Test evidence section, read against the plan's test plan (every repro, failure mode, and expected-after result it names) and the issue's repro. | Passes if the evidence re-runs the plan's repro on the branch for every failure mode the test plan names, showing for each the command and the concrete after result (the printed output, error text, or exit code, either as output lines or as a `# after:` line that states the result itself), and the after result is the plan's expected-after and differs from the before. Fails if the evidence says only "works now", "tested locally", "verified on my machine", or "matches the expected output" without showing the result; if it runs only a control or the unchanged path and never the failing case; or if any failure mode the test plan names is never re-run. | required |
| repo-checks-run | The Test evidence section's report of the repo's own tests, suite, or checks, read against what the plan's test plan and the repo facts ask to be run. | Passes if every suite or check the plan or repo asks for is reported with the command or suite named and its outcome stated (for example, "`cargo test -p ignore` passes (214 tests)" or "`go test ./...` passes"). Passes if neither the plan nor the repo asks for a suite. Fails if a requested suite or check is never mentioned, or is reported only as "tests pass" / "CI is green" with no command or suite named, or if the reported outcome is a failure. | required |
| clean-diff | Every hunk in the diff, and the commit list. | Passes if every added or removed line serves the planned change (or its tests, changelog, or a noted deviation): the fix reads on its own. Fails if the diff carries any debris: a debug print or log line, commented-out code, a dead function or unused variable, an added TODO / FIXME, formatting, whitespace, import-order, or re-indent churn on lines the fix does not otherwise need, or a hunk unrelated to the change. | required |
| template-asks-met | The repo facts' PR template and contribution asks (and live, the repo's PR template and `docs/CONTRIBUTING.md`), each ask read against the description and the diff. | Passes if every ask the repo states is met with real content: each required template section or checklist item is filled (or marked N/A with a reason), the issue link appears in the form asked ("Closes #N", "fixes #N"), and every file the repo asks for (a changelog, whatsnew, or release-note entry, generated docs) is present in the diff. Passes if the repo states no asks beyond what the PR already does. In live mode, every section of the Path Review template has real content. Fails if any stated ask is visibly ignored: a required checklist left out, the issue-link line missing, or a required changelog / whatsnew entry absent from the diff. | required |
| ai-disclosure | The repo facts' contribution and AI policy (live: `docs/CONTRIBUTING.md` and the template's disclosure section), read against the description. | Passes if, when the repo's policy or template asks for AI-use disclosure, the description discloses the AI use in the contributor's own words with everything the policy asks for (for example, the tool used and the extent of assistance). Treat every package as AI-assisted work. Passes if the repo states no AI policy and has no disclosure section. In live mode, course work always discloses, so the disclosure must be present. Fails if a stated disclosure ask is unmet or answered with boilerplate that omits what the policy asks for. | required |
| maintainer-direction | The Thread highlights (especially OWNER / MEMBER / COLLABORATOR comments), read against the diff and the description. | Passes if the PR follows every explicit maintainer direction in the thread (an approach chosen or ruled out, an API to keep, a question to answer), or addresses it in the description. Passes if the thread has no maintainer direction. Fails if the PR goes against or ignores an explicit maintainer direction without addressing it. | required |
| commit-history | The commit list: each commit's message and what it changes. | Passes if each commit message says what that commit changes. Fails if any message is uninformative ("wip", "fix", "misc cleanups", "fmt"). | preferred |
| title-says-what | The PR title, read against the diff. | Passes if the title names what changed and where, so a maintainer can tell from the title alone. Fails if the title is vague ("Fix bug", "Update code") or names a change the diff does not make. | preferred |

## Verdict rule

Accept (ready to submit) only if every required check grades pass.
Reject (hold) if any required check grades fail or unclear.

An unclear grade counts as fail. If the package does not give enough
evidence to decide a required check (for example, the test evidence
says something passed but shows no result), the PR cannot be verified
from the package, so it is not ready to submit. A check whose
condition does not apply (the repo states no asks, the thread has no
maintainer direction) grades pass, not unclear.

Preferred checks (commit-history, title-says-what) never change the
verdict. Report their grades so the contributor can fix them.

An honest shortfall can be ready. A PR that does less than the plan
passes checks 1 and 2 if the plan's deviation note records what was
left out and why, and the description restates it. Doing less than
everything is not, on its own, a reason to hold.
