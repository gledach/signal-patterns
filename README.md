# signal-patterns

Eight patterns for building tools that feed an AI agent, each one extracted from a bug
that shipped.

This is not a framework. There is no package to install and nothing to version. It is the
set of decisions that survived contact with a running system — written down because the
reasoning is the reusable part, and the code around it usually is not.

The reference implementation is **[gledach/signals](https://github.com/gledach/signals)**
— a competitive-intelligence tool that watches ~13 sources, scores and attributes what
comes back, and exposes the result over MCP. Every pattern below links to the file that
implements it and the assertion that keeps it implemented.

---

## Why these and not others

Each pattern here earned its place by failing first. Where a section says "the incident",
that is a real one, with real numbers, from a single deployment over about two months.

The bias throughout: **an AI pipeline fails silently and pleasantly.** The dashboard fills
up, nothing throws, and the system looks healthy right up until you check. Most of these
patterns are machinery for turning a quiet failure into a loud one.

---

| # | Pattern | The failure it prevents |
|---|---|---|
| 1 | [Store chokepoint](./01-store-chokepoint.md) | Schema changes with unbounded blast radius |
| 2 | [Canonical and mirror](./02-canonical-and-mirror.md) | Regeneration silently destroying human-written content |
| 3 | [Never store an uncomputed verdict](./03-uncomputed-verdicts.md) | A dead model writing guesses into permanent storage |
| 4 | [Read-only agent surface](./04-read-only-agents.md) | An agent spending your money because it found a button |
| 5 | [The coverage block](./05-coverage-block.md) | An agent reporting "nothing happened" when collection is broken |
| 6 | [Zone separation](./06-zone-separation.md) | The lethal trifecta, when your input is attacker-reachable |
| 7 | [Assertions as guardrails](./07-assertions.md) | Every pattern above quietly eroding |
| 8 | [Collect, don't write](./08-collect-dont-write.md) | Untestable sources, and shared logic wrong N ways |

---

## The shortest version

If you read nothing else:

- **One gateway to your store.** Enforced by a test, not a convention.
- **The database is canonical; files are mirrors.** Read-modify-write anything a human
  can also edit.
- **A model that failed must write nothing.** A degraded guess in permanent storage is
  worse than a halted run, because the run announces itself.
- **Agents read by default.** Every paid or state-changing action is opt-in, per
  deployment, behind a shared spend ceiling.
- **Return "I don't know" as data.** An empty result and a broken collector look identical
  to an agent, and only one of them is an answer.
- **The process holding the credential must not be the process reasoning over the input.**
- **Every rule above needs an assertion**, or it is a comment.

---

## Using this

Copy the reasoning, not the code. Most of these are ten lines and a paragraph explaining
why those ten lines are shaped that way — the paragraph is the valuable half.

Where a pattern is genuinely portable, it is marked. Most are not: the store chokepoint in
the reference implementation is 823 lines with 59 domain references, and extracting it
would mean rewriting it generically rather than lifting it. That is the honest reason this
repo is documentation rather than a library.

## Contributing

If one of these is wrong, or you have a counter-example where it cost more than it saved,
open an issue. A pattern that has never been argued with has not been tested.

---

<sub>MIT. Part of <a href="https://github.com/gledach">gledach</a> — tools that turn public
noise into structured signal.</sub>
