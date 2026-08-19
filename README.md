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

## Roadmap

See [BACKLOG.md](BACKLOG.md). The first unchecked item is the one being built.

## Planned contents

Nothing here is built yet. This table is the intended shape, and the daily loop
fills it in one item at a time.

| # | Skill | What it does |
| --- | --- | --- |
| 001 | [the-ask-first](skills/the-ask-first) | Put the point and the request in the opening, and treat everything after as support. |
| 002 | [one-page-memo](skills/one-page-memo) | The format that forces a decision: situation, complication, question, answer, and what happens next, in one page that does not cheat on margins. |
| 003 | [kill-the-filler](skills/kill-the-filler) | The catalogue of phrases that add length without meaning, with what each one is usually hiding. |
| 004 | [proposal-structure](skills/proposal-structure) | Structure a proposal around the client's problem rather than your capabilities, including how to handle the sections procurement requires and nobody reads. |
| 005 | [difficult-email](skills/difficult-email) | Write the message that delivers bad news, declines, or pushes back, without either softening it into ambiguity or making an enemy. |
| 006 | [executive-summary](skills/executive-summary) | Write the summary that works alone, because it is the only part some readers will read. |
| 007 | [voice-capture](skills/voice-capture) | Extract someone's actual register from their existing writing and apply it, rather than defaulting to a house style nobody chose. |
| 008 | [edit-for-length](skills/edit-for-length) | Cut a document by half without losing an argument, and say which arguments were lost. |
| 009 | [documentation-register](skills/documentation-register) | The voice these repositories are written in: plain declarative sentences, what a README owes the reader, and why the 'what this is not' section is the useful one. |

## Licence

MIT. See [LICENSE](LICENSE).
