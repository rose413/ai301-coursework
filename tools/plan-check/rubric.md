# Rubric: is this plan ready to post and build from?

<!--
THIS IS THE PART YOU WRITE. The skill in SKILL.md executes whatever
checks you define here (via your procedure.md). It ships empty on
purpose: the judgment is your work.

A filled rubric must contain:

1. At least one row in the checks table. Each row needs all four
   columns:
   - Check: a short name (used in the output JSON).
   - Evidence: exactly what to look at, and where in the package. Name
     the part (the plan's scope statement, the test plan read against
     the repro evidence's steps, the plan comment read against the
     thread highlights, the repo-facts block) or a location from your
     references/evidence-guide.md. "The plan" is not a source; "the
     plan's stated cause read against what the repro evidence shows"
     is.
   - Pass condition: a decision rule about the OUTCOME that someone
     else could apply and get your answer. Judge the thing itself (is
     this one bounded change? could a stranger start executing it?),
     never the write-up's shape (how many sections it has, how long it
     is, whether it uses headings). Structure-shaped checks are what
     make graders disagree with themselves.
   - Weight: `required` (a fail here holds the package) or `preferred`
     (never changes the verdict).

2. A verdict rule below the table: how the check grades combine into
   accept (ready) or reject (hold), including how `unclear` is
   treated. The verdict space is binary. If you write no rule for
   `unclear`, the skill treats it as fail.

Cover what actually gets bad plans posted. The lecture named the
failure families: the diagnosis ignores or contradicts the reproduced
evidence, the change is unbounded (scope creep), the plan targets the
symptom while the evidence points at the cause, a stranger could not
start executing it, the test plan proves nothing observable, the
unknowns are dressed up as certainty, and the comment ignores what the
thread or the repo's stated conventions ask. A rubric that ignores a
family will fail eval packages designed around that family.
-->

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| diagnosis | The candidate plan's stated cause (its Cause / Diagnosis part), read against every step, control, and timing in the Repro evidence block. If the cause comes from a thread comment, read that comment against the repro evidence too. | Passes if the stated cause explains every behavior the repro evidence shows, including its controls, and no repro step contradicts it. Fails if any repro step or control shows the behavior happening where the cause says it cannot (for example, the bug still appears with the blamed component removed or bypassed), or if the cause is taken from the thread without being checked against the repro. Also fails if the change patches the place where the symptom appears while the cause is upstream of it. | required |
| scope | The candidate plan's Scope / Change part: its in-scope list, its not-in-scope line, and the files or code areas it names. Read the not-in-scope line against the Repro evidence. | Passes if the plan is one bounded change: it names the files or code areas it will change, says what it will not touch, and every change listed serves the diagnosed cause. Leaving part of the issue for later passes if the plan says so explicitly. Fails if the plan adds unrelated work ("while I'm here" renames, refactors, docs passes, migrations, new flags), or if the repro evidence points at something the plan lists as out of scope. | required |
| executable | The candidate plan's Change / Approach part: the files, functions, or code sites named and the steps of the approach. | Passes if a stranger could start the work without asking the author anything: it names where the change goes (a file, function, or specific code site) and what the change is. Fails if the location is a guess ("somewhere in the editor code", "poke around"), the approach is a goal rather than a change ("make undo work"), or key unknowns are left for the build to discover. | required |
| test | The candidate plan's Test / Test plan part, read against the Repro evidence block's steps and its Expected / Actual lines. | Passes if the test re-runs the repro (or an equivalent concrete check) and states the observable result expected after the fix, one that differs from the repro's Actual result, so a stranger can tell pass from fail. Fails if the test is only "run the test suite and check nothing regresses", only restates the goal ("undo works", "see if it works"), or names no expected-after result. | required |
| comment | The candidate plan comment, read against the candidate plan, the Thread highlights (especially OWNER / MEMBER / COLLABORATOR comments), and the Repo facts block's contribution policy and AI policy. | Passes if the comment promises only what the plan contains and presents as fact only what the plan and repro support, AND it does not go against or ignore a maintainer signal in the thread (an API to keep, an approach ruled out, a "working as intended" or "the fix is hard" note) without addressing it, AND it meets the repo's stated contribution rules (for example, an AI-use disclosure when the policy asks for one). An empty thread and no stated policy put nothing further on the comment. Fails if any of these does not hold. | required |

## Verdict rule

Accept (ready to post and build from) only if every required check
grades pass. Reject (hold) if any required check grades fail. An
unclear grade (?) counts as fail: if the package does not give enough
evidence to decide a check, the plan is not ready to build from, so
the package is held. Preferred checks, if any are added, never change
the verdict.
