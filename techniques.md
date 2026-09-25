# What is now possible, and what it costs

LLM capabilities relevant to a monitoring pipeline, as of September 2026, with the cost
levers ranked by what they actually save. Sources at the bottom.

**The headline for anyone running a classifier on a schedule:** the three biggest savings
are not model choice. They are, in order — **use the batch API** (flat 50%), **cache the
prefix** (reads at 0.1×), and **stop paying for output you do not read**. Stacked, the
first two reach ~95% on the repeated portion of the work.

---

## 1. Batch APIs — a flat 50%, and almost nobody uses them

Non-real-time work qualifies for a **flat 50% discount** across providers. A classifier
running on a six-hourly cron is the textbook case: nobody is waiting for the response, and
a few hours of latency changes nothing.

If your pipeline is scheduled, **this is free money you are not taking.** The only work is
restructuring submission from "call per item" to "submit a file, poll, read results", and
handling the fact that results come back **in any order** — key by your own id, never by
position.

Caching and batching **stack**.

## 2. Prompt caching — reads at one tenth

Universal in 2026, with different ergonomics per provider:

| Provider | How | Read | Write |
|---|---|---|---|
| OpenAI | automatic from 1,024 tokens | 0.1× | 1.25× (30-min retention from GPT-5.6) |
| Anthropic | explicit `cache_control` marker | 0.1× | 1.25× at 5-min default, 2× for 1-hour |
| Gemini | implicit on 2.5+ | no guaranteed discount | — |

**The ordering rule is the whole technique:** tools → system prompt → documents → history
→ the live query. Anything that changes per request must come *after* the last cache
breakpoint. Leave a timestamp or a session id in the cacheable prefix and you pay the write
premium every call for a cache that never hits — strictly worse than not caching.

Verify rather than assume: check `cache_read_input_tokens` (Anthropic) or `cached_tokens`
(OpenAI). **Zero reads across repeated calls means a silent invalidator**, and because
writes cost more than plain input, a cache that never reads is a net loss.

## 3. Output tokens are the expensive side

Output is priced 4–5× input on most cards, so the cheapest thing you can do is ask for
less of it. *If you only need a classification, ask for a single token, not a paragraph.*

**The trap this creates:** a *reasoning* model on a classification task bills its
deliberation as output. Measured on one real pipeline — a reasoning model answering "is
this a product launch" against a fixed schema emitted **1,905 output tokens and took 41.7
seconds per item**. 146 items took 101 minutes. A non-reasoning instruct model of the same
family was ~4× cheaper and dramatically faster at the same job.

Corollary: **"flash" does not mean cheap.** One provider's flash tier at $0.75/$3.75 costs
*more* on an output-heavy workload than another provider's pro tier at $0.65/$1.30.
Compare the output column first.

## 4. Structured outputs

Every major provider now constrains responses to a schema. Two benefits, and the second is
the one people miss:

- Parsing stops being a guessing game.
- **It caps output length**, which is the cost lever above.

For a classifier, a schema with an enum and a confidence float is both more reliable and
cheaper than "reply with JSON like this".

## 5. Routing

Send the easy 80% to a small model and escalate only what is ambiguous. This is the
standard advice and it is real — but do it **last**. It adds a decision point that can be
wrong, and the three levers above are unconditional.

## 6. Instrument tokens, not requests

The reason bills surprise people: **teams measure requests when cost varies per token.**
Two calls to the same endpoint can differ by an order of magnitude. Log per call: model,
input tokens, output tokens, cost, and *which code path ran*. Without per-call attribution
you are optimising blind, and you cannot answer "what would this run cost?" before running
it.

Worth logging explicitly: the **method** that produced the answer. A fallback path that
emits the same shape as the real path, with no field recording which ran, is how a pipeline
silently fills with guesses.

## 7. Semantic deduplication before the model

The cheapest call is the one you do not make. Deduplicate at ingestion — exact-hash first,
then near-duplicate — so the same press release syndicated across ten outlets is classified
once. In a news pipeline this is routinely the largest single saving, and it is not an LLM
technique at all.

---

## Applying this to a watcher

In rough order of return:

1. **Batch the scheduled classifier.** 50%, no quality change.
2. **Dedup before classifying**, within the batch as well as against the store.
3. **Cache the system prompt and few-shots**; verify reads are non-zero.
4. **Use an instruct model, not a reasoning model**, for fixed-schema triage.
5. **Constrain the output schema** so you stop buying prose you discard.
6. **Log tokens per call, with the code path**, and build a pre-flight estimate from it.
7. **Route** only once the above are done.

---

## Sources

- [Batch Processing With LLMs in 2026 — ProjectSupply](https://projectsupply.in/blog/batch-processing-llms-cost-effective-2026)
- [Prompt Caching in 2026: Cut LLM Costs, Keep Quality — Digital Applied](https://www.digitalapplied.com/blog/prompt-caching-2026-cut-llm-costs-engineering-guide) — per-provider read/write multipliers, prefix ordering
- [LLM Cost Optimization: Caching, Batching & Routing — DataNorth](https://datanorth.ai/blog/llm-cost-optimization-prompt-caching-batching-routing)
- [LLM Cost Reduction: 12 Strategies — NeuralTrust](https://neuraltrust.ai/blog/llm-cost-reduction-guide) — cache/route/compress
- [Reduce LLM API Costs 60% — Kunal Ganglani](https://www.kunalganglani.com/blog/reduce-llm-api-costs-production) — token-level attribution
