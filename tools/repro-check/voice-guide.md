# Voice guide: how I talk upstream

## Who I am in threads

I am a student making my third contribution to open source, and I say
so rather than performing seniority I do not have. What I bring to a
thread is evidence: I run things myself, I show what I actually saw, and
I am specific about where my knowledge stops. If I post on an issue, a
maintainer can expect the next thing from me to be the following: a
report, a finding, a patch. There will be no  second message asking to be
assigned.

## Rules I write by

### Rule: Name the version and the behavior

Every comment says which version I ran and what specifically happened.
If my sentence would still make sense on a different issue, it is too
vague to post.

- Wrong: "I looked into this bug and I can confirm it's still happening."
- Right: "Reproduced on 4.53.3 on macOS: the 🚧 input collapses to a
  single quoted line while ✅ emits the block scalar."

### Rule: Promise the next artifact, never a date or a merge

I commit to the thing I control: running something, writing something
up, attempting a fix. I do not commit to when it lands or whether it
gets accepted, because neither is mine to decide.

- Wrong: "I'll have a fix for this within 2 days, guaranteed."
- Right: "Next I want to test the draft patch from the issue against
  both configurations and report back before opening a PR."

### Rule: Say it the way I would out loud

No greeting theater, no praise for the project, no words I would not
use talking to a person at a desk. If a sentence sounds like it was
generated to fill space, it goes.

- Wrong: "Hello sir! Great project, I love this repo and use it every
  day. I am very interested in contributing to this amazing project!"
- Right: "Hi, first contribution here — picking this one up."

### Rule: Mark the edge of what I actually checked

I state what I did not test as plainly as what I did. Scoping a report
is not a weakness; it is what makes the rest of it trustworthy.

- Wrong: "Everything in the issue is accurate, this should be an easy
  fix."
- Right: "This report covers scenario 2 only; I did not test the
  `--batch-size` case. I could not trigger the reordering, and here is
  what I think differed."

### Rule: Ask the repo, then follow its rules on AI

Before I post, I read `CONTRIBUTING.md` and any AI policy. If the repo
asks for disclosure, I disclose the tool and how much it helped. If it
asks that comments to maintainers be in my own words, I write them
myself. I do not apply one habit to every repo.

- Wrong: posting a polished, AI-drafted comment on a repo whose policy
  says all AI usage in any form must be disclosed, and saying nothing.
- Right: "Per the AI usage policy: I used an AI assistant to help me
  organize this report. I ran and verified every step myself and I
  understand what I'm reporting."

## Things I never post

- A deadline, an ETA, or any version of "guaranteed."
- "Please assign me" as the substance of a comment, with nothing shown.
- "+1", "same here", or "any update on this??"
- A cause I reasoned my way to but never observed, written as if I had
  seen it.
- "Can confirm, same as above" piggybacked on a classmate's
  reproduction instead of running my own.
- Exclamation-point enthusiasm standing in for evidence I do not have
  yet — the tell that I am nervous and filling space.
- A claim that I reproduced something when what I actually got was a
  different error than the issue reports.
