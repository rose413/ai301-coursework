# Evidence guide: where evidence lives in a plan package

<!--
THIS IS THE PART YOU WRITE (second week running: the judgment files
stay in your hands). The skill uses this guide as its map: for every
kind of evidence a rubric check names, this file says WHERE to find it
in a plan package and WHAT GOOD LOOKS LIKE when you do.

Under each family heading below, write:

- Where it lives: the exact places to look. In an eval bundle (which
  section of the package: the issue context, the repro-evidence block,
  the candidate plan's scope statement or test plan, the plan comment,
  the repo-facts block). In live mode (where on GitHub or in the
  draft: the issue thread, the student's posted repro comment, the
  repo's docs, the draft plan and comment).
- What good looks like: one or two sentences someone else could apply.
  Prefer observable conditions ("the stated cause cites behavior the
  repro evidence actually shows") over adjectives ("diagnosis is
  solid").

A rubric check whose evidence this guide cannot locate is a check
nobody else can execute, and this week that cuts twice: your
procedure.md tells the skill WHEN to gather each family, and this
guide tells it WHERE. Write the map you wish your executor had.
-->

## Diagnosis and grounding

**Where it lives.** Eval: the cause is the first part of
`## Candidate plan` (labelled "Cause:" or "### Diagnosis"; in an
unstructured plan, the sentence that says why the bug happens). The
behavior it must explain is in `## Repro evidence`: the numbered
steps, any control runs (a component disabled or bypassed), timings,
and the Expected / Actual lines. If the plan credits the thread ("as
identified in this thread"), that claim is in `## Thread highlights`.
Live: the cause is in the draft `plan.md`; the repro evidence is the
student's posted repro comment on the issue (or the house repro pack
as quoted in the drafts).

**What good looks like.** Every repro step and control is consistent
with the cause, and the change lands where the cause is, not where
the symptom shows. Bad: the repro shows the bug with the blamed part
taken out of the loop (calib-03: the plan blames the pager's key
bindings, but repro step 3 is just as slow with no pager at all and
step 2 is fast in the same pager with colors off). Bad: the cause
is a commenter's claim copied without checking it against the repro.

## Scope

**Where it lives.** Eval: `## Candidate plan`, in the "Change:" or
"### Scope" part: the "In:" / "In scope:" list, the "Out:" / "Not in
scope:" line, and the files or code areas named there and in
"### Changes" / "### Approach". Live: the same parts of `plan.md`.

**What good looks like.** One change, with named files or areas, a
stated not-in-scope line, and every listed change serving the cause
(calib-01: one callback in `sync_controller.go`; out: how push status
is computed and other views). Leaving part of the issue for later is
fine when the plan says so. Bad: extra work that the cause doesn't
need (renames, refactors, docs passes, config migrations, new flags).
Bad: a not-in-scope line that excludes what the repro points at.

## Executability

**Where it lives.** Eval: `## Candidate plan`, in the "Change:",
"### Changes", or "### Approach" part: the file paths, functions, and
code sites named, and the numbered steps. Live: the same parts of
`plan.md`.

**What good looks like.** A stranger could open the named file and
start: there is a specific location (file path, function, callback)
and a specific change ("add the commits context to the post-push
refresh scope"). Bad (calib-02): "undo is probably handled somewhere
in the editor code", "poke around this weekend", "make undo work".
These are guesses and goals, not changes.

## Test plan

**Where it lives.** Eval: the "Test:" or "### Test plan" part of
`## Candidate plan`, read against the steps and the Expected / Actual
lines of `## Repro evidence`. Live: the test plan in `plan.md`, read
against the student's posted repro comment.

**What good looks like.** The test re-runs the repro steps (or an
equivalent concrete check) and states the result expected after the
fix, which differs from the repro's Actual line (calib-01: "repro
steps above; at step 3 the color must flip without leaving the
view"). Bad: "run the full test suite and make sure nothing
regresses" (calib-04): the suite passes today, so it proves nothing
about the bug. Bad: "undo works after toggling" (calib-02): no steps,
no observable result to check.

## Honesty

**Where it lives.** Eval: hedges and certainty in the cause and
approach of `## Candidate plan` (words like "probably", "maybe",
"traced", "as identified"), any risks or unknowns the plan lists, and
factual claims in `## Candidate plan comment`. Live: the same in
`plan.md` and the draft comment, plus the Deviations section of
`plan.md` after the build.

**What good looks like.** Unknowns are named as unknowns and do not
sit where the approach should be. Claims of certainty ("I traced
this") are backed by the repro or the code site named in the plan.
Bad: the comment says "I traced this" when the plan just repeats a
thread commenter's claim. Bad: an approach that leaves the cause for
the build to discover. Mid-build changes are recorded in the plan's
Deviations section, not only in the diff.

## Comms

**Where it lives.** Eval: `## Candidate plan comment`, read against
three things: the candidate plan (what it actually promises),
`## Thread highlights` (comments by OWNER, MEMBER, or COLLABORATOR
are maintainer signals), and `## Repo facts` (the "contribution
policy" line, including any AI-use policy and review-bandwidth
notes). Live: the draft comment, the live issue thread, and the
repo's CONTRIBUTING.md / AI policy files.

**What good looks like.** The comment promises only what the plan
contains, responds to what maintainers already said, and follows the
repo's rules (calib-04's comment addresses the owner's "the fix is
hard" note, stays out of the area the owner flagged, and gives the
AI-use disclosure the policy asks for). calib-01's comment notes the
review-bandwidth line in CONTRIBUTING. Bad: boilerplate ("I'm going
to fix it, wish me luck!") that ignores the thread. Bad: proposing an
approach a maintainer ruled out without saying why. Bad: missing an
AI disclosure the policy asks for, or promising more than the plan
holds.
