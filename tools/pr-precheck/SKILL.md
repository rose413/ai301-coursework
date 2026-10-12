---
name: pr-precheck
description: Grade a PR package (a candidate pull request read against the plan it claims to implement and the issue that plan belongs to) and decide whether it is ready to submit. Use when checking your own branch, draft PR title, and description before opening the pull request, or when grading an eval package bundle.
---

# pr-precheck: rubric-driven PR grading

## The question

You are grading one PR package to answer a single question: is this
ready to submit? A PR package is a candidate pull request (its title,
description, commits, diff, and test evidence) read against the plan
it claims to implement, that plan's deviation notes, the issue the plan
belongs to, and the repo's stated standards (its PR template and
contribution policy). Grade exactly one package per run. Do not answer
any other question: you are not reviewing code style, suggesting a
better fix, re-grading the plan, or predicting whether a maintainer
will merge. You do not answer from gut feel: you answer by executing
`procedure.md`, which applies the checks in `rubric.md` to evidence
gathered per `references/evidence-guide.md`.

## Inputs and modes

Decide the mode first. If you were handed a package bundle (a markdown
file from the eval set, such as `eval/packages/pkg-07.md`, or its text
inlined by the harness), you are in eval mode. Otherwise you are in
live mode.

**Live mode**: the student's own PR, checked before they open it.
Read these inputs, and only these:

1. **The files the invocation names.** Read every file the student's
   run command points you at: their `plan.md` (default
   `beat-1-sandbox/unit-3/plan.md`), including its Deviations section;
   `test_evidence.md`, their captured test evidence (before/after
   output and the repo's own checks); and `pr_draft.md`, their draft
   PR title and description. Read the whole file each time; do not
   grade from a remembered copy.
2. **The branch's diff.** Run `git diff main...HEAD` (three dots) from
   the student's working copy of their fork, on their `fix/...` branch.
   That output is the diff: everything the branch changes relative to
   `main`. Also run `git diff --stat main...HEAD` for the list of
   changed files, and `git log --oneline main..HEAD` for the commits.
   Do not grade uncommitted working-tree changes: they are not in the
   PR. If you see uncommitted changes, mention them in the summary.
3. **The issue's thread.** Fetch the issue the plan belongs to (via
   `gh issue view <number> --repo <scoped repo> --comments`, the GitHub
   API, or the web): the issue body, every comment, and any maintainer
   direction in it.
4. **The repo's standards.** Read the repo's PR template
   (`.github/PULL_REQUEST_TEMPLATE.md` or wherever the repo keeps it)
   and `docs/CONTRIBUTING.md`, from the working copy or from the
   scoped repo on GitHub. Their stated asks, including AI-use
   disclosure, are part of the bar.

A house-chain student (one the instructor routed to the house issue)
reads the house plan and the house repro pack in place of their own
plan and repro. The same checks grade the same things there.

The package is what these inputs contain. Do not read other files in
the student's working directory as evidence, and do not count a claim
the student makes in the conversation unless it appears in the draft
or the captured evidence.

**Eval mode**: grade one package bundle. The bundle is the whole
world: its repo-facts block, issue, thread highlights, plan context,
and candidate PR (title, description, commits, diff, test evidence).
Use only the bundle text as evidence. Do not fetch anything, run git,
or read any other file, including the gold labels and other packages.
Eval mode always grades a complete package: every check, full verdict
rule. The `item` in the JSON is the bundle id (for example `pkg-07`).

### When you refuse to grade

Stop without a verdict, and say why, when any of these is true:

- `rubric.md` has no checks or no verdict rule, or `procedure.md` has
  no steps (see Component reads). This applies in both modes.
- Live mode: `scope.md`'s `Repo:` line is still an unfilled
  placeholder, or the PR would target a repo the scope does not name
  (see The scope seam).
- Live mode: there is nothing to grade. You cannot find the plan, or
  `git diff main...HEAD` is empty (the branch has no committed change).
  Tell the student which input is missing and how to supply it. Do not
  invent a plan or grade an empty diff.
- You are asked to grade more than one package in one run, or to
  answer a different question. Grade one package, or decline.

Any other missing input (no draft description, no captured test
output, no template in the repo) is not a refusal. Grade it: the
checks that need that evidence grade its absence, as the rubric and
Grading discipline direct.

## The scope seam (live mode only)

In live mode, read `scope.md` in this skill directory before anything
else, before the plan or the diff.

- If its `Repo:` line still carries the placeholder (brackets, or the
  "replace with your section's Path Review repo" text with no real
  `owner/name`), stop without grading. Tell the student to fill the
  `Repo:` line in `scope.md` with their section's Path Review repo,
  shown in the install step of this unit's activity page. Never guess a
  scope.
- Check the student's PR against the scoped repo: their fork's
  `upstream` remote, or the issue they named, must be the repo on the
  `Repo:` line. If the PR would target any other repository, refuse to
  grade it and say which repo the scope allows.
- Apply the house rules `scope.md` lists as part of how you read
  evidence: the PR comes from the student's own fork on a
  `fix/<issue-number>-<slug>` branch against `main`; one PR per issue;
  every template section gets real content; a classmate's PR on the
  same issue does not block this one. If a rubric check reads a house
  rule, grade it there. Otherwise report a broken house rule in the
  summary.

In eval mode, ignore `scope.md` entirely. Do not read it.

## The voice seam (live mode only)

In live mode, after the scope, read `voice-guide.md`: the student's own
rules for how they write upstream. Hold the outgoing text, the draft PR
title and description, against every rule in it. For each rule the
draft breaks, report it in the summary: quote the rule and the line of
the draft that breaks it, and say what would fix it. If the guide is
still an unfilled template, say so in the summary and skip the voice
pass. Do not refuse over it.

The voice guide never changes the verdict on its own. It is the
student's personal register, not a standard the maintainer stated. Only
a rubric check that explicitly reads `voice-guide.md` can carry it into
a grade.

In eval mode, ignore `voice-guide.md` entirely. Voice is personal and
carries no gold labels. Universal communication-quality checks live in
the rubric.

## Component reads

Read the components in this order, then grade:

1. `rubric.md`: the checks table (each row's name, evidence, pass
   condition, and weight: `required` checks gate the verdict,
   `preferred` checks never change it) and the verdict rule below it.
   Use the check names exactly as written. They become the `name`
   fields in the JSON.
2. `references/evidence-guide.md`: the map of where each evidence
   family lives (plan fidelity, test evidence, diff quality, standards
   and comms) in a bundle and in live mode, and what good looks like
   there. When a check names an evidence family, look where this guide
   says.
3. `procedure.md`: the operating steps. Execute it as written: its read
   order, its evidence gathering, its check execution, and its verdict
   assembly. Do not reorder it, skip steps, or add steps of your own.

Where the procedure is silent on a step you need (for example, it never
says where to gather evidence for a check the rubric names), do the
minimum the rubric's own evidence column directs, and report the gap in
the summary under a "Procedure gaps" line. Do not invent a step and do
not paper over it. A procedure gap is feedback the student needs.

Before grading, check that `rubric.md` and `procedure.md` have content.
HTML comments (`<!-- ... -->`) and headings are template, not content.
If the rubric's checks table has no filled row, or it has no verdict
rule, or `procedure.md` has no written steps under its headings, stop.
Say which file is empty and that this tool cannot grade without a
rubric and a procedure. Emit no checks and no verdict. An empty tool
that invents checks at runtime produces noise that looks like judgment.

## Verdict and output

The verdict space is binary: `accept` means ready to submit (open the
PR), and `reject` means hold (fix first). There is no third verdict, no
"accept with reservations", and no score. Reservations belong in check
evidence lines and the summary, not in the verdict.

Before the JSON, write a short readable summary:

- one line per check: its name, its grade, and the evidence that
  decided it;
- the deciding check (the check the verdict turned on, picked by
  `procedure.md`'s verdict assembly), quoted;
- in live mode, any voice-guide breaks and any house-rule notes;
- any procedure gaps.

Then end the reply with the fenced JSON block below, filled in. It must
be valid JSON, contain one entry per rubric check in rubric order, and
be the last thing in the reply, with nothing after it. Each `evidence`
value is one line naming the fact or quote that decided the grade.
`item` is the PR URL in live mode (or the fork branch, if the PR is not
open yet) and the bundle id in eval mode.

```json
{
  "item": "<PR URL or bundle id>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear",
     "evidence": "<one line: the fact or quote that decided it>"}
  ],
  "verdict": "accept|reject"
}
```

## Grading discipline

- **Evidence first.** Never grade a check without naming the fact or
  quote that decided it: a diff line, a plan line, an output line, a
  template section. "Looks fine" is not evidence.
- **Grade the thing, not the polish.** A terse, complete PR can be
  ready, and a long, confident one can be hiding drift. Read the diff
  against the plan and its deviation notes, the evidence against the
  test plan and the repo's checks, and the description against the
  diff. Never grade length, formatting, or tone.
- **Honest shortfalls can pass.** A PR that discloses what it does not
  do, with the deviation recorded in the plan and restated in the
  description, can be ready. Less than everything is not, on its own, a
  hold. A deviation that exists only in the diff is silent drift.
- **The rubric decides, not you.** If a check passes by its stated
  condition but feels wrong, it still passes. Note the tension in the
  summary if you want. The fix belongs in the rubric, not in the run.
- **The procedure decides how, not you.** Follow `procedure.md` as
  written, and report its gaps instead of papering over them.
- **Unclear defaults to fail.** Treat `unclear` as the rubric's verdict
  rule directs. If the rule does not say, treat `unclear` as `fail`: a
  claim you cannot verify from the package is a failing claim, and a PR
  you cannot verify is not ready to submit.
