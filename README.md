# signal-patterns

**How to build tools that watch the public web and feed the result to an AI agent.**

What to watch, what it costs, how to build it, and the engineering decisions that only
become obvious after something has broken.

Not a framework. There is no package to install and nothing to version — the reasoning is
the reusable part, and the code around it usually is not. The reference implementation is
**[gledach/signals](https://github.com/gledach/signals)**, which watches ~13 sources across
a market and exposes the result over MCP.

---

## Start here

| | |
|---|---|
| **[What is watchable →](./sources.md)** | A catalogue of public sources ranked by lead time, cost and precision. Certificate logs run 20–30 days ahead; changelog pages change on 73% of monitors within 90 days; news is the last place anything appears. Includes what to refuse. |
| **[What it costs →](./techniques.md)** | LLM capabilities as of September 2026 and the cost levers ranked by return. The top three are not model choice: batch APIs (flat 50%), prefix caching (reads at 0.1×), and not buying output you discard. |
| **[Building a watcher →](./building-a-watcher.md)** | The shape, and the decisions that are easy to get wrong — hashId choice, state, cadence, failure behaviour. |
| **[Being consumable by agents →](./agent-integration.md)** | MCP so an agent can query you, Agent Skills so it knows how to use you, and why permission must not scale with capability. |
| **[Engineering patterns →](./patterns/)** | Eight decisions, each extracted from a bug that shipped. |

---

## The thesis

A company is only as visible as what it exposes publicly, and **the useful signals are the
ones it exposes before it means to.** Ordered by lead time:

```
infrastructure  →  hiring  →  documentation  →  changelog  →  marketing  →  press
   (weeks)         (months,      (hours)         (hours)      (at launch)   (after)
                    vague)
```

Most tools start at the press end. That is the last place anything appears, and it is
where every competitor is already looking.

The second half of the thesis is about the consumer. **Hand the result to an agent, not to
a dashboard** — a dashboard needs someone to open it, and the thing you want is an answer
to a question you have not thought to ask yet. That shifts the requirements: structured
output, stable ids, and the ability to say *"I am not in a position to answer"* rather than
returning an empty list that reads as "nothing happened".

That half is now settled by two open standards rather than by integrations. **MCP** is how
an agent queries you; **Agent Skills** is how it learns to use you. The two most-used open
personal-agent runtimes of 2026 both speak both, and both are local-first — so a tool built
as a local process with a queryable store fits them with no work, and a tool built as a
hosted product fits neither. See [agent-integration.md](./agent-integration.md).

---

## The eight patterns

Each earned its place by failing first. Where one says "the incident", that is a real one,
with real numbers, from a single deployment over about two months.

The bias throughout: **an AI pipeline fails silently and pleasantly.** The dashboard fills
up, nothing throws, and it looks healthy right until you check. Most of these are machinery
for turning a quiet failure into a loud one.

| # | Pattern | The failure it prevents |
|---|---|---|
| 1 | [Store chokepoint](./patterns/01-store-chokepoint.md) | Schema changes with unbounded blast radius |
| 2 | [Canonical and mirror](./patterns/02-canonical-and-mirror.md) | Regeneration silently destroying human-written content |
| 3 | [Never store an uncomputed verdict](./patterns/03-uncomputed-verdicts.md) | A dead model writing guesses into permanent storage |
| 4 | [Read-only agent surface](./patterns/04-read-only-agents.md) | An agent spending your money because it found a button |
| 5 | [The coverage block](./patterns/05-coverage-block.md) | An agent reporting "nothing happened" when collection is broken |
| 6 | [Zone separation](./patterns/06-zone-separation.md) | The lethal trifecta, when your input is attacker-reachable |
| 7 | [Assertions as guardrails](./patterns/07-assertions.md) | Every pattern above quietly eroding |
| 8 | [Collect, don't write](./patterns/08-collect-dont-write.md) | Untestable sources, and shared logic wrong N ways |

---

## The shortest version

- **Watch what leaks, not what is announced.** Certificates, docs and changelogs beat press releases.
- **Cadence is the cost dial**, not model choice. A per-subject job on the wrong tier cost one deployment ~$209/month for no added freshness.
- **Batch and cache before you optimise anything else.** Flat 50%, then reads at a tenth.
- **A failed model writes nothing.** A halted run announces itself; a run that writes guesses does not.
- **Return "I don't know" as data.** Empty and broken look identical to an agent.
- **Agents read by default.** Every paid or state-changing action opt-in, behind a shared ceiling — and **permission must not scale with capability**.
- **A skill is a supply chain.** 341 malicious skills were found among ~13,000 in one 2026 marketplace audit.
- **Every rule above needs an assertion**, or it is a comment.

---

## Contributing

If one of these is wrong, or you have a counter-example where it cost more than it saved,
open an issue. A pattern that has never been argued with has not been tested. New sources
worth watching are equally welcome — especially ones with a lead time and a measured
precision attached.

---

<sub>MIT. Part of <a href="https://github.com/gledach">gledach</a> — tools that turn public
noise into structured signal.</sub>
