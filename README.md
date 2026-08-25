# writing-agent

Agents and skills for business writing that says something.

Most business writing is long because writing short is harder, and
vague because vagueness is safer. The result reads as competent and commits to
nothing, and the reader has to reconstruct the point from context.

This repository holds the agents and skills for the opposite: the point in the
first paragraph, the ask stated as an ask, the reasoning available for anyone who
wants it and out of the way of anyone who does not.

It also holds the register these repositories are written in, which is the same
discipline applied to documentation.

## What this is not

It does not write in your voice unless it has been given your
voice. A skill that produces confident prose in nobody's voice is the problem it
is meant to solve.

It is not a grammar checker.

It does not make a weak argument persuasive. Where the argument is the problem,
it says so.

## How to use this

These are skills for Claude, not a command-line tool. There is nothing to
install and nothing to import — you describe the work and the matching skill
fires on its own.

**In Claude Code or Cowork**, once the skills are on your machine:

```bash
bash ~/Desktop/agent/_setup/sync-skills.sh
```

That clones every agent repository and links its `skills/` into `~/.claude/skills`,
so they are available in every session and every folder. Re-run it whenever one of
these repositories ships something new — it pulls rather than re-clones.

Then simply ask. Each skill's `description` frontmatter is written to match how
the request actually gets phrased, in English or Turkish, so you do not name the
skill and generally should not have to think about which one applies.

**If nothing fires**, that is a defect in the skill rather than in how you
asked. The description was written for the wrong phrasing. Say what you asked
and what you expected, and it gets fixed — that feedback is more valuable than
working around it.

**What is actually built** is listed under Contents below and in the Done
section of [BACKLOG.md](BACKLOG.md). Everything under Queue is planned and does
not exist yet.

## Layout

```
agents/
  <name>.md           one specialist, its brief and its boundaries
skills/
  <name>/
    SKILL.md          the instruction, with triggering description frontmatter
    scripts/          only where deterministic code beats instruction
examples/
  <name>/             worked example on real input, with the output committed
```

`examples/` is empty so far.

## Roadmap

See [BACKLOG.md](BACKLOG.md). The first unchecked item is the one being built.

## Contents

| Skill | What it does |
| --- | --- |
| [cold-email](skills/cold-email) | Write a cold email that earns a reply, with the pretext stated rather than manufactured. |
| [copywriting](skills/copywriting) | Write or rewrite marketing copy for a page, starting from what the reader has to decide. |
| [marketing-psychology](skills/marketing-psychology) | Apply behavioural principles to a marketing decision, with the ethical limit of each one stated. |
| [uret](skills/uret) | Generate images and video through a provider API, saving each output next to the prompt that made it. Needs an API key, and states the cost before a paid video call. |

| Agent | What it does |
| --- | --- |
| [icerik-yazari](agents/icerik-yazari.md) | Writes text that will be published. |

These arrived already written and in daily use, rather than being built against the queue below — which is why most carry no item number. Some have Turkish bodies: they were written in the language they are used in, and translating them is a queue item rather than a blocker.

Everything still under Queue in [BACKLOG.md](BACKLOG.md) does not exist
yet.
## Licence

MIT. See [LICENSE](LICENSE).
