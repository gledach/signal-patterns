# Being consumable by agents

How a tool makes itself usable by an AI agent, as of September 2026. Two open standards do
almost all the work, and the interesting decisions are about permission rather than
protocol. Sources at the bottom.

**The short version:** implement **MCP** so an agent can *query* you, ship **Agent Skills**
so it knows *how to use* you, and keep both **read-only by default** — because a more
capable runtime must not mean more permission.

---

## The landscape, briefly

The two most-used open personal-agent runtimes of 2026 are **Hermes Agent** (Nous Research,
released February 2026 — 140k+ GitHub stars in three months, and #1 on OpenRouter at ~224B
tokens/day) and **OpenClaw** (released November 2025 as Clawdbot, renamed January 2026 —
250k+ stars, ~2M monthly actives, ~186B tokens/day).

What matters for a tool author is that they converged on the same two things:

| | Hermes Agent | OpenClaw |
|---|---|---|
| Query surface | MCP | MCP |
| Procedure format | Agent Skills (`SKILL.md`) | Agent Skills (`SKILL.md`) |
| Posture | local-first, MIT, no telemetry | local-first, self-hosted, model-agnostic |
| Scheduling | natural-language cron per profile | "heartbeat" polling cycle |
| Memory | tiered — `MEMORY.md`, SQLite FTS5, episodic archive | appended JSONL logs |
| Skills | self-generated from execution, then refined | human-written, static after install, distributed via a marketplace |

**Both are local-first.** If your tool assumes it is the centre of a hosted product, it
does not fit either. If it assumes it is a local process with a queryable store, it fits
both with no work.

The divergence worth noting is self-improvement: one runtime writes and refines its own
skills from successful executions, the other treats skills as static human-authored
artefacts. If you publish a skill, assume it may be *rewritten* by the runtime consuming it.

## 1. MCP — so an agent can ask

Model Context Protocol over stdio is line-delimited JSON-RPC with no client-specific
behaviour. One server implementation satisfies every compliant runtime; "does it work with
X" is answered by X's documentation, not by yours.

Three things a tool author should get right:

**Locate your own project.** The server will be launched from an arbitrary working
directory. Resolve paths from the module, not from `cwd`, or you ship a server that works
when *you* run it.

**Return coverage, not just rows.** See [pattern 5](./patterns/05-coverage-block.md). An
agent that reads `matched: 0` reports "nothing happened" without suspicion.

**Expose resources as well as tools.** Tools are verbs; resources are nouns an agent can
browse. A tool-only server forces an agent to guess what exists.

## 2. Agent Skills — so an agent knows how

[Agent Skills](https://agentskills.io) is a folder containing a `SKILL.md` with YAML
frontmatter. Released by Anthropic as an open standard, it is now implemented by roughly
forty-five clients — editors, terminal agents and personal-agent runtimes alike.

```
skill-name/
├── SKILL.md          # required: frontmatter + instructions
├── scripts/          # optional: executable code
├── references/       # optional: detail loaded on demand
└── assets/           # optional: templates
```

```yaml
---
name: skill-name          # 1–64 chars, lowercase + single hyphens, MUST match the directory
description: What it does, and when to use it.   # 1–1024 chars
license: MIT              # optional
---
```

### Progressive disclosure is the whole design

1. **Discovery** — at startup the client reads *only* `name` and `description` for every skill.
2. **Activation** — the full body loads when a task matches the description.
3. **Execution** — `scripts/` and `references/` load only if used.

**So the description is not documentation. It is the entire basis on which your skill is
chosen**, and the one part guaranteed to be in context. "Helps with PDFs" loses to
"Extracts text and tables from PDF files, fills forms, merges PDFs. Use when working with
PDF documents or when the user mentions forms or document extraction."

Say **what it does and when to use it**. Include the words someone would actually type.

### The failure mode is silence

A `SKILL.md` without frontmatter is not an error. It simply never appears. The reference
implementation shipped nine skills for months that worked in the single client they were
written for and were invisible in every other — discovered only by reading the spec.

Keep the body under ~500 lines; it loads whole on activation. Push detail into
`references/`.

## 3. Permission does not scale with capability

This is the part standards do not give you.

An agent exploring a new tool **calls everything once to see what it does.** That is
reasonable behaviour, and it means a capable runtime plus a permissive tool is a bill, or
worse. So:

- **Read by default.** Every paid or state-changing action opt-in, per deployment.
- **One shared ceiling** across scheduler, CLI and agent, so no caller can starve another.
- **The policy does not vary by client.** A more capable runtime does not get more
  permission — if it did, your security model would be "whoever asks nicest".

Full reasoning in [pattern 4](./patterns/04-read-only-agents.md).

## 4. A skill is a supply chain

A 2026 audit of the largest public skill marketplace found **341 malicious skills among
roughly 13,000**. Security analyses of these runtimes describe a three-tier attack surface
— cognition, execution, interaction — with real vulnerabilities at each; one runtime
disclosed three CVEs in April 2026, including a path traversal in a messaging adapter and
an API-server authentication flaw.

**A skill is instructions an agent will follow, from a file you did not write.** That is a
dependency, and it deserves the scrutiny you give an npm package rather than the scrutiny
you give a config entry. Read one before installing it.

It also sharpens [pattern 6](./patterns/06-zone-separation.md). If your tool ingests
attacker-reachable content — search results, RSS, an inbox — and an agent with a
marketplace skill can also write to your store, every leg of the lethal trifecta is present
and two of them arrived from outside.

## 5. What this means for a monitoring tool specifically

A collection tool is close to the ideal agent citizen, because it is **read-heavy, local,
and answers questions the agent did not know to ask**. Four things make it land:

- **Stable ids.** An agent that cannot re-reference a result cannot follow up.
- **Structured output with provenance.** A claim with no source is unusable to something
  that will be asked "how do you know".
- **An honest empty answer.** See pattern 5. This is the one most tools get wrong.
- **A scheduler the agent does not own.** Both runtimes have their own cadence mechanism.
  If your tool also collects on a schedule, say which is authoritative — two schedulers
  over one store is a duplicate-writes problem waiting to happen.

---

## Sources

- [Agent Skills — specification](https://agentskills.io/specification) — frontmatter fields, progressive disclosure, size guidance
- [Agent Skills — overview and client list](https://agentskills.io)
- [Hermes Agent](https://hermes-agent.org/) · [NVIDIA on Hermes](https://blogs.nvidia.com/blog/rtx-ai-garage-hermes-agent-dgx-spark/)
- [OpenClaw](https://openclaw.ai/) · [docs.openclaw.ai/tools/skills](https://docs.openclaw.ai/tools/skills)
- [Hermes Agent vs OpenClaw (2026)](https://hackernoon.com/hermes-agent-vs-openclaw-which-ai-agent-framework-wins-in-2026) — skill formats, marketplace audit, scheduling
- [A Security Analysis of the OpenClaw AI Agent Framework](https://arxiv.org/pdf/2603.27517) · [Security of OpenClaw Agents](https://arxiv.org/pdf/2605.25435)
