# Rubric: is this reproduction package ready to post?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| env-recorded | The repro report's environment record (the `Environment:` line or equivalent), read against the asks listed in the repo-facts block's `bug reports:` line. | The report names the tool/project version it ran and the platform it ran on (OS or the equivalent the repo's template asks for, e.g. browser, driver, install method). A record that is one terse line still passes if the values are there. Fail when any of those values is absent from the package, even if the artifact looks convincing. | required |
| steps-followable | The repro report's steps section, read as a stranger with a clean machine and nothing but this package. | The steps give a starting state and the exact commands, inputs, or UI actions needed to reach the failure, and every input they depend on (config, file contents, sample data, repro link) is either included in the package or fetchable from the public repo. Fail when a step depends on something only the author has (a private repo, an unshared config, "my project") or omits a setting the issue's behavior depends on. | required |
| artifact-backs-claim | The artifacts in the repro report (output excerpts, logs, transcripts, screenshots described) read against the report's own `Actual:` / result statement. | The package contains at least one artifact produced by running the steps, and that artifact displays the outcome the report says it shows. Fail when there is no artifact at all, when the artifact only shows that the tool started or ran, or when the artifact shows a different result than the sentence describing it claims (including an `Expected:`/`Actual:` pair stated backwards from the output). An artifact showing that nothing went wrong passes when the report's claim is that it could not reproduce. | required |
| target-faithful | The steps and environment record read against two things: the condition the issue identifies as producing the failure, and the version the issue targets (the issue body, the reporter's stated version, and any version confirmation the repo's template requires). | Both must hold. (a) Trigger: the run preserves the condition the issue names as causing the failure — the same input shape, syntax, and precondition (exactly one custom header, the offset-from-end range, the tuple passed to the same call). Changing *how* the run is performed while preserving that condition is fine — running offline, scripting the setup, a fresh playground — especially when the report says so. What fails is changing the condition itself, so that a different code path is exercised: a swapped operator, an altered expression, a skipped precondition. (b) Version: the version or build tested is the one the issue targets, or the report names the difference in its own words. Fail when an older or newer version is tested and the delta is never called out. | required |
| claim-specific-honest | The candidate claim comment, read against the issue it is posted on. | The comment names something true of this issue in particular (the version, the specific behavior, a finding from the thread) such that it could not be pasted onto a different issue unchanged, and whatever it commits to is a next artifact the author controls — an investigation, a report, a fix attempt. Fail on interchangeable boilerplate ("great project", "please assign me"), on a guaranteed outcome or deadline ("fixed within 2 days"), and on a bare +1 or self-assignment with no stated intent. | required |
| policy-satisfied | The `contribution policy` entry in the repo-facts block, read against both candidate comments. | Read what the policy actually requires of issue comments, then check the comments against that. If it requires AI use to be disclosed, pass only when a comment names the AI assistance and its extent; treat the package as AI-assisted work, so a missing disclosure is a real failure and not an unknown. If it requires comments to maintainers to be in the contributor's own words, pass when the comments read as a person's own writing rather than generated filler. If it states no requirement that touches issue comments, pass. | required |
| control-run | The repro report's artifacts, looking for a second run that differs from the failing one in one variable (the working input, the unaffected version, the English-first case). | A control run is shown alongside the failing run, isolating what makes the difference. Absence never holds a package; a control is what turns a correct report into a persuasive one. | preferred |

## Verdict rule

`accept` when every `required` check grades `pass`.

`reject` when any `required` check grades `fail` or `unclear`. Evidence
that is absent from the package is evidence a stranger cannot check, so
`unclear` counts as `fail` — the package is held, not failed forever.

`preferred` checks never change the verdict. Record their grades and say
so in the summary.

Two consequences worth stating, because they are where graders drift:

- A reproduction is not required. An honest cannot-reproduce that
  records its environment, gives followable steps, shows the artifacts
  of the attempt, and says plainly what it could not trigger passes
  every required check and is `accept`. Reporting a negative result
  faithfully is a contribution.
- Polish is not evidence. A long, well-formatted, confident report whose
  artifact shows a different failure than the issue describes fails
  `artifact-backs-claim` or `target-faithful` and is `reject`. A
  six-line report with the values in it is `accept`.
