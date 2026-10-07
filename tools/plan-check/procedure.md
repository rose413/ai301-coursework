# Procedure: how this skill grades a plan package

<!--
THIS IS THE PART YOU WRITE, and it is a new kind of part. Weeks 1 and
2, SKILL.md carried a numbered workflow and you only wrote judgment
files. This week the workflow is gone from the frame: SKILL.md says
"execute procedure.md", and these are the operating steps you author.
The machinery is in your hands now.

Your operator swap is the design brief. When your executor stalled
because your rubric said WHAT to decide but not HOW to find the
evidence, that was a procedure gap. This file is where those gaps get
closed: a complete procedure lets someone who has never seen a plan
package before (a groupmate, or the skill itself) grade one exactly the
way you would.

Under each stage heading below, write the concrete steps for that
stage. The one-line note under each heading says what a complete
procedure must decide there. Write steps, not intentions: "read the
repro evidence before the plan, and note what behavior it pins down"
is a step; "understand the context" is a wish.
-->

## Read order

<!-- What gets read, in what order, before any check is graded, and
what to note down from each part while reading. A complete procedure
decides the order (issue first? repro evidence first?) and says why
the order matters for the checks that come later. -->
Read the package parts in this order, and take the notes listed with
each step before moving on. Do not grade any check while reading.

1. **Repro evidence block first.** Note: the exact steps, each
   observed result, any controls (runs with a component disabled or
   bypassed), any timings, and the Expected and Actual lines. This is
   the baseline every later part is held to, so read it before any
   claim about the cause can shape how you read it.
2. **Issue.** Note the reported behavior and the expected behavior.
   Confirm the repro reproduces the same behavior. Note any part of
   the issue the repro does not cover.
3. **Thread highlights.** Note each maintainer signal (comments by
   OWNER, MEMBER, or COLLABORATOR): APIs or behaviors to keep,
   approaches ruled out, "working as intended" or "the fix is hard"
   notes. Note each claimed root cause from non-maintainers as a
   claim, not as a fact.
4. **Repo facts block.** Note the contribution policy, any AI-use
   policy (disclosure asks, human-written comment rules), and any
   review-bandwidth notes.
5. **Candidate plan.** Find its stated cause, its in-scope and
   not-in-scope lines, the files or code sites it names, its approach
   steps, and its test plan. Note the exact wording of each; these
   are what get quoted later.
6. **Candidate plan comment last.** Note every promise it makes and
   every statement it presents as fact. It is read last because it is
   graded against everything above it.

## Evidence gathering

<!-- For each evidence family your rubric's checks name, the concrete
gathering move: which part of the package (or, live, which page or
thread location per your evidence guide) to pull the fact from, and
what to record. A complete procedure leaves no check whose evidence an
executor would have to hunt for. -->
For each check in `rubric.md`, pull the evidence listed below before
grading anything. `references/evidence-guide.md` says where each kind
of evidence lives and what good looks like.

1. **diagnosis:** Write down the plan's stated cause in one line,
   quoted. Then list every repro step, control, and timing from step
   1 of Read order. For each one, record whether the cause explains
   it, contradicts it, or does not address it. If the cause names a
   source in the thread, record that, and check that claim against
   the repro the same way. Record where in the code path the change
   lands and where the cause sits.
2. **scope:** Copy the plan's in-scope list, its not-in-scope line,
   and the files or areas it names. For each listed change, record
   whether it serves the stated cause. Record anything the repro
   evidence points at that sits on the not-in-scope line. Record any
   part of the issue the plan leaves for later, and whether the plan
   says so.
3. **executable:** Record each place the plan says the change goes
   (file, function, code site) and each approach step. Mark each as
   concrete (named) or vague (guessed, "somewhere", "figure out").
4. **test:** Copy the test plan. Record (a) which repro steps it
   re-runs, (b) the expected-after result it states, and (c) whether
   that result differs from the repro's Actual line.
5. **comment:** List each promise and factual claim in the comment
   and record whether the plan contains it. List each maintainer
   signal from step 3 of Read order and record whether the comment or
   plan respects or addresses it. List each repo-facts rule from step
   4 of Read order that applies to a comment and record whether the
   comment meets it.

## Check execution

<!-- How one check runs against gathered evidence: in what order the
checks execute, what an executor does when evidence for a check is
genuinely absent, and when a check may be graded without re-reading
the whole package. A complete procedure makes two executors grade the
same package the same way. -->
1. Run the checks in this order: diagnosis, scope, executable, test,
   comment. Diagnosis goes first because scope and test are judged
   against the cause it settles. Comment goes last because it is
   graded against the plan.
2. Grade every check, even after one fails. The output must list all
   of them.
3. For each check, apply its pass condition from `rubric.md` to the
   evidence you recorded in Evidence gathering, word for word. Do not
   add conditions the rubric does not state, and do not let a check
   pass because the plan "feels" right.
4. Grade only from what you recorded. Re-read a package part only if
   your notes do not answer the pass condition.
5. If the evidence a check needs is missing from the package (for
   example, the plan has no test plan, or no stated cause), grade that
   check **fail** and record "missing: <what>" as its evidence. A plan
   that leaves out a part cannot pass a check on that part.
6. Use **unclear** only when the evidence is present but the package
   does not let you decide either way (for example, two readings of
   the plan's wording lead to opposite grades). Record the two
   readings as the evidence.
7. Judge the content, not the formatting. A short plan with no
   headings can pass every check; a long plan with every section can
   fail.
8. For each check, write one evidence line: quote the plan or comment
   text that decided it and, for a fail, the repro, thread, or
   repo-facts line it conflicts with.

## Verdict assembly

<!-- How the per-check grades become the final accept or reject:
apply your rubric's verdict rule, state how unclear grades enter it,
and say what gets quoted in the output for the deciding check. A
complete procedure produces the same verdict from the same grades,
every time. -->
1. Count the grades on required checks.
2. If every required check is **pass**, the verdict is `accept`.
3. If any required check is **fail** or **unclear**, the verdict is
   `reject`. Unclear counts as fail, per the rubric's verdict rule.
4. Preferred checks never change the verdict.
5. Name the deciding check: the first failing required check in the
   execution order (or "all required checks pass" for an accept). In
   the summary, quote the submission's own line for it next to the
   line it conflicts with (for example, the plan's cause next to the
   repro step that contradicts it).
6. Emit the JSON block from SKILL.md as the last thing in the output,
   with one entry per check in execution order and the verdict from
   steps 2 to 3.

