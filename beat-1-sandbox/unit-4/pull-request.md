# Unit 4 — Test and Submit

Path: `beat-1-sandbox/unit-4/pull-request.md`

Record of the pull request you opened against the Path Review repo, and of the evaluation
runs that produced `eval-run.txt`. This file is graded at the path above; a copy kept
anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your pull request

**Pull request**

https://github.com/rose413/pathreview-ai301-fa26-s3/pull/1

**Branch**

fix/38-auth-middleware-tests

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

1. Run 1: graded with the same skill files as run 2. All 19 packages that returned a
   verdict matched gold. `pkg-05` returned no grader result in this run, so it did not
   agree. That makes it 19/20.
2. Run 2 (final, saved with `--save-run`): **19/20** (`agreement: 19/20 scored items  (bar:
   18/20: PASS)`). Categories: `clear-accept 6/7  not-tested 4/4  silent-drift 4/4
   standards-wall 2/2  unreviewable 3/3`. The one miss was `pkg-05`, which my rubric
   rejected and gold accepts. This matches the committed `eval-run.txt`.

**Package analysis**

`pkg-05` (nushell/nushell#18848, "Keep same-name keybindings that target different keys
when merging config"). My rubric's verdict was **reject**; the gold label is **accept**.
The eval line reads `pkg-05  clear-accept  accept  reject  NO  failed: repro-re-run`.

Every other required check passed. The diff touches only `keybindings.rs` and its tests,
the description's claims match the diff, the description answers both CONTRIBUTOR
comments in the thread, and `cargo test -p nu-protocol` is named with an outcome. The only
failure was `repro-re-run`. The plan's test plan names two failure modes:

> re-run the issue's script, expect both `atuin` rows; re-run with two same-name same-key
> bindings, expect one row plus a warning

The first is shown with the command and before/after tables. The second is only
asserted:

> Same-key redefine prints the one-time warning.

My check requires "showing for each the command and the concrete after result" for every
failure mode the test plan names. The grader therefore graded it fail with this note:
"Plan test item 2 (same-name same-key, expect one row plus warning) has no command or
output, only 'Same-key redefine prints the one-time warning.'"

Gold reads the package as a whole. The same-key case is covered by a test in the diff,
`same_name_same_key_replaces_with_warning`, which asserts
`assert_eq!(merged.iter().filter(|b| b.name == "atuin").count(), 1);` and
`assert_eq!(warnings.len(), 1);`. The evidence also says "`cargo test -p nu-protocol`
passes (312 tests)". So the second failure mode is verified, but by a test that passes,
not by a pasted re-run. My rubric only counts a shown re-run, so it held a PR that a
maintainer would accept.

**Check rationale**

> | repro-re-run | The Test evidence section, read against the plan's test plan (every
> repro, failure mode, and expected-after result it names) and the issue's repro. | Passes
> if the evidence re-runs the plan's repro on the branch for every failure mode the test
> plan names, showing for each the command and the concrete after result (the printed
> output, error text, or exit code, either as output lines or as a `# after:` line that
> states the result itself), and the after result is the plan's expected-after and differs
> from the before. Fails if the evidence says only "works now", "tested locally", "verified
> on my machine", or "matches the expected output" without showing the result; if it runs
> only a control or the unchanged path and never the failing case; or if any failure mode
> the test plan names is never re-run. | required |

This check exists to catch the not-tested family: PRs whose test evidence is a judgment
("Tested locally, works now, all tests pass.") instead of a result. I wrote it to grade
the *outcome* (is there an observable after result for each failure mode?), not the
write-up's shape. The fail list names the exact phrases that look like evidence but are
not: "works now", "verified on my machine", "matches the expected output".

Two parts of it are deliberate choices. First, I accept a `# after:` line "that states
the result itself" as well as pasted output lines. calib-01 shows its after result as a short
`# after:` line followed by `parsed OK`, not a full output block (my evidence guide cites it
as the example). A check that demanded a pasted transcript would have failed that good
package. Second, I require *every* failure mode the test plan
names, not just the headline repro, and I added "only a control or the unchanged path"
as an explicit fail. Re-running a control that already worked proves nothing about the
fix, and a check that accepted any one re-run would let that through. I rejected letting
a passing test suite stand in for a re-run: "tests pass" is exactly the claim the
not-tested packages hide behind.

**Trade-offs**

The case I accept this check will miss is `pkg-05`. Its requirement is strict: every
failure mode needs a shown command and after result. That holds a PR whose second
failure mode is proven by a unit test in the diff (`same_name_same_key_replaces_with_warning`)
plus a reported passing suite, but is only asserted in prose in the test evidence. That
one strictness is the entire gap between my 19/20 and a 20/20: `pkg-05` failed on
`repro-re-run` alone, with every other required check passing.

I kept it strict anyway. In the final run, `repro-re-run` failed on all four not-tested
packages (`pkg-04`, `pkg-07`, `pkg-10`, `pkg-14`) and they scored `not-tested 4/4`. The
loosening that would rescue `pkg-05` ("a failure mode counts as re-run if a test in the
diff asserts it and the suite is reported passing") is the same move those packages rely
on: pointing at tests instead of showing the failing case fixed. I'd rather hold a good
PR and ask for one more pasted command than submit one that was never actually re-run.
The fix for the contributor is cheap, and the grader's note says what to paste. I did not
edit the rubric between run 1 and run 2: `rubric.md` was last written before either run
started, so the two runs differ only in grader behaviour, not in the check.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/pr-precheck/`.
