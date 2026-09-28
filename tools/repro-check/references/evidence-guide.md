# Evidence guide: where proof lives in a reproduction package

## Environment

**Where it lives.** In an eval bundle: the `Environment:` line at the
top of the `## Candidate repro report` section. Read it against the
`bug reports:` entry in the `## Repo facts` block, which lists exactly
what that repo's template asks reporters for, and against the version
named in the `## Issue` body (reporters usually state theirs in the
body or a versions block). In live mode: the environment line in your
draft report, against the repo's `.github/ISSUE_TEMPLATE/` bug form and
the issue's own stated version; the repo's Releases page tells you what
"latest" currently means.

**What good looks like.** It names the version of the thing under test
and the platform it ran on, plus whatever else that repo's template
specifically asks for — install method for yq and fd, driver for
minikube, browser for a web project, `pd.show_versions()` output for
pandas. The values are what matter, not the formatting: one comma-
separated line carries as much proof as a table. The failure to look
for is absence, not brevity — a report that never says which version or
which OS it ran on cannot be placed by a maintainer, no matter how good
its log looks, because the same log means different things on different
builds.

## Steps

**Where it lives.** In an eval bundle: the `Steps:` block of the repro
report, plus any setup shown inside the artifact fences (the `mkdir`,
the `printf` that writes the input file, the config file quoted before
the run). Read it against the issue's own reproduction instructions in
the `## Issue` section. In live mode: your draft's steps, against the
issue body's repro section and the project's README for how the tool is
normally installed and invoked.

**What good looks like.** A stranger with a clean machine can start at
step one and arrive at the failure without asking you anything. That
means a starting state (a fresh clone, an empty directory, a named
release, a fresh playground), the exact commands or actions in the
order run, and the actual content of every input they depend on — the
config file quoted, the sample document written out by a command in the
transcript, the playground link included. The specific failure to watch
for is a step that only the author can perform: a private monorepo, an
unshared config, "my project," an internal build. Those make the report
unre-runnable even when it is completely honest. Also watch for a step
that silently drops a condition the issue depends on, such as omitting
the driver on an issue that only happens with one driver.

## Behavior shown

**Where it lives.** In an eval bundle: the fenced blocks in the repro
report — terminal transcripts, log excerpts, produced output, described
screenshots — read against two things: the report's own `Actual:`
sentence, and the behavior the `## Issue` section describes (the error
text, the exit code, the wrong value, the visual symptom). In live
mode: the output you paste into your draft, against the issue body and
any maintainer comment that pins down what correct output should be.

**What good looks like.** The artifact was produced by running the
steps, and it displays the outcome the sentence next to it says it
displays. Read the artifact first and the prose second, then check they
agree: an exit code, an error class, a message, a rendered value.
Three failures live here and each is quieter than the last. The first
is no artifact at all — a confident diagnosis, a root cause named, a
"guaranteed reproducible," with nothing shown. The second is an
artifact that proves only that the tool is installed and running: a
version banner, a session list, a window that opened. The third is the
hardest and the one polish hides: an artifact that shows a real failure
which is not the issue's failure — a graceful argument-validation error
where the issue reports a panic, a compile error where the issue
reports a path-expression error, garbled output with the program still
alive where the issue reports a crash. When the artifact and the issue
name different failures, the artifact is the fact and the prose is the
claim.

## Honesty

**Where it lives.** The seam between the report's claim sentences (the
`Result:`, `Expected:`, and `Actual:` lines, and any summary at the
top) and the artifacts directly above or below them. In live mode, the
same seam in your draft — plus the gap between what you actually ran
tonight and what you are about to imply you ran.

**What good looks like.** Every sentence stays inside what the evidence
supports, and the report says out loud where the evidence stops. A
report that scopes itself ("this report is about scenario 2 only; I did
not test scenario 1") is stronger, not weaker. A cannot-reproduce that
shows the attempt, states the negative result plainly, and names what
probably differed — uniform name lengths, a 2 MiB `ARG_MAX`, Linux and
zsh where the reporter had macOS and fish — is fully honest proof and
should read as ready. What fails here is claiming more than was shown:
a cause asserted from reading the code rather than observed, "I
verified this race condition" with no transcript, a deviation from the
issue's version or trigger that goes unmentioned, or an `Expected:`
line written backwards from what the artifact actually printed. The
test is simple: cover the prose and look only at the artifacts, then
ask what they prove. Anything the prose adds beyond that is a claim,
and a claim needs to be marked as one.

## Comms

**Where it lives.** In an eval bundle: the `## Candidate claim comment`
section, read against the `## Issue` title and body, the `## Thread
highlights` list, and — for what the repo permits — the `contribution
policy` entry in the `## Repo facts` block. In live mode: your draft
comment, against the issue thread, `CONTRIBUTING.md`, any `AI_POLICY.md`
or AI-usage section, `AGENTS.md`, and the Code of Conduct.

**What good looks like.** The comment could not be pasted onto another
issue without becoming false. It names the version and the specific
behavior, or picks up something from the thread — a maintainer's spec
for correct output, a linked upstream fix, a pointer at the function
where the failure lives. What it commits to is the next artifact the
author actually controls: a repro report, an investigation, a fix
attempt, a report-back. Being new is fine to say and often helps.

Two distinct things fail here. The first is boilerplate over-promising:
generic praise, "kindly assign it to me," "reserved for me," a
guaranteed fix in a stated number of days, a bare +1. These fail even
when the repro report beneath them is excellent, because the comment is
what the maintainer reads first and it is the part addressed to a
person.

The second is the repo's own rules about AI, and these differ, so read
the policy before judging rather than applying one habit everywhere.
Some repos state nothing. Some are permissive: generative AI is welcome
and you are responsible for what you submit, with no disclosure asked.
Some require the comment to be in your own human words and will hide
generated comments, which a plainly human-voiced comment satisfies with
no disclosure needed. Some ask for disclosure only on pull requests and
say so explicitly. And some require that all AI usage in any form be
disclosed, naming the tool and the extent of the help — for those, a
comment that does not disclose fails, and since the work in this course
is AI-assisted, a missing disclosure is a real absence rather than
something unknowable from the package.
