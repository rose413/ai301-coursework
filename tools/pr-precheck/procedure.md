# Procedure: how this tool grades a PR package

## Read order

Read the package parts in this order, and take the notes listed with
each step before moving on. Do not grade any check while reading. In
eval mode every part is a section of the bundle. In live mode, each
step names where the part comes from; `references/evidence-guide.md`
has the full map.

1. **Issue and thread highlights.** Note the reported behavior and
   the expected behavior. Note each maintainer direction (comments by
   OWNER, MEMBER, or COLLABORATOR): an approach chosen or ruled out, an
   API or behavior to keep, a question asked of the contributor.
   (Live: `gh issue view <number> --comments` on the scoped repo.)
2. **Plan context.** Copy, word for word: the plan's Files / scope
   line (every file or folder it names), its not-in-scope line, each
   step it promises, its test plan (every repro, failure mode, and
   expected-after result it names, numbered), and any deviation note
   or deferral. Also note the repro evidence's before result. (Live:
   `plan.md`, including its Deviations section; a house-chain student
   uses the house plan and house repro pack.) The plan is read before
   the diff and the description so that it is the yardstick for both,
   not something the description's summary can shape.
3. **Repo facts.** List every ask the repo states, one per line:
   each PR template section or checklist item, the issue-link form
   ("Closes #N", "fixes #N"), required files (changelog, whatsnew,
   release notes), suites or checks to run, and the AI policy and what
   a disclosure must include. Note "no stated asks" or "no AI policy"
   where that is the case. (Live: the repo's PR template and
   `docs/CONTRIBUTING.md`.)
4. **Diff.** List every changed file, and for each file every hunk,
   in one line each: what the hunk changes. Mark each hunk that adds a
   debug print or log, commented-out code, a dead function or unused
   variable, a TODO / FIXME, or formatting, whitespace, import-order,
   or re-indent churn. (Live: `git diff --stat main...HEAD` for the
   file list, `git diff main...HEAD` for the hunks.)
5. **Commits.** List each commit's message and the files it touches.
   (Live: `git log --oneline main..HEAD`.)
6. **Test evidence.** List each command shown, and next to it the
   result shown for it (output lines, or a `# before:` / `# after:`
   line), quoted. List each statement about the repo's suite or checks
   (the command or suite named, and the outcome stated). (Live:
   `test_evidence.md`.)
7. **Title and description last.** List every claim the title,
   description, and checklist make, one per line, quoted: fidelity
   claims ("implements the plan exactly"), things added ("tests for
   X", "changelog added", "docs updated"), things not changed or not
   added ("no options added"), deferrals, and the AI disclosure. They
   are read last because every one is graded against the parts above.
   (Live: `pr_draft.md`.)

## Evidence gathering

For each check in `rubric.md`, build the side-by-side listed below
from your Read order notes before grading anything. Record each
pairing as a row: the left item, the matching right item (or "none"),
and yes / no.

1. **diff-matches-plan: diff files against plan scope, then plan
   steps against the diff.**
   - For each changed file from Read order step 4: does the plan's
     Files line name it or its folder ("a changelog entry" names the
     changelog file), is it a test, fixture, or changelog file for the
     planned change, or does a deviation note cover it? Record yes and
     which, or no.
   - For each hunk in a named file: does it serve the planned change?
     Record any hunk that adds an option, flag, config key, or rename,
     and whether the plan asks for it, rules it out, or does not
     mention it.
   - For each step the plan promises from Read order step 2: record
     the hunk that delivers it, or "none". For each "none", record
     whether a deviation note covers it.
2. **description-matches-diff: description claims against the diff.**
   For each claim from Read order step 7, record the diff line that
   proves it, or what is missing. A claim that something was added
   needs a hunk that adds it. A claim that something was not changed
   or not added is a yes only if no hunk does it, and a fidelity claim
   ("exactly the plan", "no changes beyond it") is a no if step 1
   recorded any no. Then, for each deviation or deferral in the plan,
   record whether the description restates it.
3. **repro-re-run: test evidence against the plan's test plan.** For
   each numbered item in the plan's test plan, record the command in
   the test evidence that re-runs it, or "none". For each one found,
   record the after result it shows, quoted, and mark it concrete (it
   states the output, error text, or exit code) or a judgment only
   ("works", "matches the expected output", "verified"). Record
   whether the after result is the plan's expected-after and differs
   from the before. Record whether the command runs the failing case
   or only a control or the unchanged path.
4. **repo-checks-run: suite reports against what is asked.** List
   each suite or check that the plan's test plan or the repo facts ask
   for. For each, record the test evidence line that reports it, the
   command or suite it names, and the outcome stated, or "none".
5. **clean-diff: hunks against the change.** Copy every hunk marked
   as debris in Read order step 4, with its file and what it is. List
   any hunk step 1 recorded as not serving the planned change.
6. **template-asks-met: repo asks against the description and
   diff.** For each ask from Read order step 3, record where the PR
   meets it (the description line or the diff file) or "none". A
   section marked N/A with a reason counts as met. In live mode, list
   each section of the Path Review template and record whether it has
   real content.
7. **ai-disclosure: AI policy against the description.** Record
   whether the repo asks for disclosure (policy or template section);
   in live mode it always does. Record the description's disclosure
   line, quoted, or "none", and each item the policy says a disclosure
   must include, with whether the line includes it.
8. **maintainer-direction: thread against the PR.** For each
   maintainer direction from Read order step 1, record whether the diff
   follows it, the description addresses it, or neither.
9. **commit-history:** Record each commit message from Read order
   step 5 and whether it says what the commit changes.
10. **title-says-what:** Record the title and whether it names what
    changed and where, and whether the diff makes that change.

## Check execution

1. Run the checks in `rubric.md` order: diff-matches-plan,
   description-matches-diff, repro-re-run, repo-checks-run,
   clean-diff, template-asks-met, ai-disclosure, maintainer-direction,
   commit-history, title-says-what. diff-matches-plan goes first
   because description-matches-diff grades fidelity claims against
   its result.
2. Grade every check, even after one fails. The output must list all
   of them.
3. For each check, apply its pass condition from `rubric.md` to the
   side-by-side rows you recorded in Evidence gathering. A single no
   row in a required pairing fails the check unless the pass condition
   itself excuses it (a deviation note, an N/A with a reason). Do not
   add conditions the rubric does not state, and do not let a check
   pass because the PR "feels" ready or the description sounds sure.
4. Grade only from what you recorded. Re-read a package part only if
   your rows do not answer the pass condition.
5. If a check's condition does not apply (the repo states no asks, no
   AI policy, no maintainer direction, no requested suite), grade it
   **pass** and record "n/a: <why>" as its evidence.
6. If evidence a check needs is missing from the package (no test
   evidence section, no description, a planned step with no hunk),
   grade that check **fail** and record "missing: <what>" as its
   evidence. Missing is not unclear.
7. Use **unclear** only when the evidence is present but the package
   does not let you decide either way (for example, a hunk that could
   be read as part of the planned change or as unrelated work). Record
   the two readings as the evidence.
8. Judge the content, not the formatting. A terse PR can pass every
   check; a long, confident one with a perfect checklist can fail.
9. For each check, write one evidence line: quote the package line
   that decided it and, for a fail, the line it conflicts with (for
   example, the diff file next to the plan's Files line, or the
   description's claim next to the diff that contradicts it).

## Verdict assembly

1. Look at the grades on the required checks (the first eight).
2. If every required check is **pass**, the verdict is `accept`.
3. If any required check is **fail** or **unclear**, the verdict is
   `reject`. Unclear counts as fail, per the rubric's verdict rule.
4. Preferred checks (commit-history, title-says-what) never change
   the verdict.
5. Name the deciding check: the first required check, in rubric
   order, that graded fail or unclear (or "all required checks pass"
   for an accept). In the summary, quote the package's own line for it
   next to the line it conflicts with. If other required checks also
   failed, list them after the deciding check so the contributor sees
   everything to fix.
6. Emit the JSON block from SKILL.md as the last thing in the output,
   with one entry per check in rubric order and the verdict from steps
   2 and 3.
