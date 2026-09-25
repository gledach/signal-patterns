# Building a watcher

How to add a source, and the decisions that are easy to get wrong. The reference
implementation is [gledach/signals](https://github.com/gledach/signals); the shape below
is general.

---

## 0. Decide whether it is worth watching

Before any code, answer three questions. If you cannot, the watcher will produce volume
rather than signal.

1. **What decision would this change?** "Interesting" is not a decision. "We reprice" or
   "we add this to the objection handler" is.
2. **How early does it move, and how precise is it?** See [sources.md](./sources.md).
   Early-and-vague and late-and-certain are both useful; *late and vague* is not.
3. **What does one item cost to judge?** Volume × per-item cost is your monthly bill.
   A source producing 500 items a day at $0.006 each is $900 a month.

## 1. The shape

**A collector collects.** It returns candidates and its next state. It does not classify,
score, deduplicate, store or notify — all of that is identical across sources and belongs
in one shared runner.

```js
export default defineCollector({
  id: 'example',
  cadence: '6h',
  rateLimit: { perMinute: 60 },

  async collect({ company, state, signal, fetchImpl = fetch }) {
    const res = await fetchImpl(url, { signal });   // honour `signal` — it is the deadline
    if (!res.ok) throw new Error(`example ${res.status}`);   // throw; the runner isolates it
    return {
      items: [{ hashId, sourceKind: 'example', title, link, pubDate, summary }],
      nextState: { cursor },
    };
  },
});
```

`fetchImpl = fetch` is the most important line. **A function that returns its findings is
testable with no database, no network and no API key.** Watchers that write directly need
all three to test at all, which is why they are almost never tested.

## 2. Choose the `hashId` carefully

It is the deduplication key and it is permanent. A row written under a bad id is never
re-offered and never corrected.

- **Use the source's own stable identifier** — `hn:<objectID>`, `gh:<releaseId>`.
- **Never hash mutable text.** A retitled article becomes a second row.
- **Decide collisions deliberately.** If one story mentions two tracked companies, do you
  want two rows or one? Either is defensible; silence is not.

## 3. State, not timestamps

Return a `nextState` the runner hands back next time — a cursor, an etag, a high-water
mark. Prefer what the source gives you over "everything since `lastRun`", because a missed
run silently creates a gap and a clock skew silently creates duplicates.

Keep state in the database, not on disk. A scheduled job on ephemeral infrastructure loses
its filesystem on every deploy; a watcher whose "have I seen this" check is a file
existence test re-fetches everything, for ever, and looks fine while doing it.

## 4. Cadence is the cost dial

**Anything making one model call per tracked subject does not belong on a frequent tier.**

Measured on one deployment: a per-company synthesis job sitting on a six-hourly schedule
instead of a daily one cost **~$209/month** on its own, against a documented budget of
$15–25. It produced no extra freshness, because the underlying corpus moved by a handful
of rows between runs.

Ask what the source's own timescale is. Certificate logs move in minutes; answer-engine
visibility moves on model releases. Matching cadence to that is the single biggest lever
you have, and it is free.

## 5. Classify cheaply

See [techniques.md](./techniques.md). The short version:

- Use an **instruct** model, not a reasoning model, for fixed-schema triage.
- **Batch** it — scheduled work qualifies for a flat 50% discount.
- **Cache** the system prompt and few-shots; verify reads are non-zero.
- **Constrain the output schema** so you are not buying prose you discard.

## 6. Fail correctly

Three rules, each from a real incident:

- **A failed model writes nothing.** If the provider is down, halt. A degraded guess in
  permanent storage is worse than a halted run, because the run announces itself.
- **Isolate per item and per source.** One bad row must not drop the remaining sources.
  Put the `try/catch` around the write, not just the fetch.
- **Do not retry a self-inflicted timeout on a paid call.** The model was still
  generating; every attempt is billed.

## 7. Make an empty result distinguishable from a broken one

An empty result and a dead collector look identical to whatever reads your output — and a
consumer that reads `matched: 0` will report "nothing happened" without suspicion.

Return coverage alongside the rows: when was this last collected, which subjects are
actually covered, and **is an empty answer trustworthy right now**. Derive health from the
newest record rather than from the scheduler's log, so a hand-run deployment is not
reported as dead.

## 8. Before you ship it

- [ ] Tests pass with **no network, no database, no key**
- [ ] Cadence justified against the source's own timescale
- [ ] Per-item cost known and written down
- [ ] `hashId` stable, collisions decided
- [ ] State in the database, not on disk
- [ ] No brand or market vocabulary hardcoded — derive it from config
- [ ] An assertion enforces whichever of the above would be silent if broken

---

**Next:** [sources.md](./sources.md) for what to watch ·
[techniques.md](./techniques.md) for what it costs ·
[patterns/](./patterns/) for the engineering decisions underneath
