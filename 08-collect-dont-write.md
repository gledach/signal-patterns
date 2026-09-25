# 8. Collect, do not write

**A source returns candidates. Everything after "here is a candidate" is shared and lives
once.**

## The problem

Nine collectors, nine bespoke shapes. Each hand-rolled the same five steps around its own
fetch: deduplicate, classify, score, store, notify. Nine chances to get shared logic
subtly wrong, and adding a tenth source meant copying 200 lines.

It also made two bugs possible that no single collector looked responsible for:

- **Per-item deduplication.** One round trip per candidate to learn "seen it", when most of
  a run is duplicates.
- **Isolation at the wrong level.** The `try/catch` wrapped the fetch, not the writes - so a
  throw from the store rejected the whole run and dropped every remaining source.

## The split

```js
export default defineCollector({
  id: 'example',
  async collect({ company, state, signal, fetchImpl = fetch }) {
    return { items: [ { hashId, sourceKind, title } ], nextState };
  },
});
```

A collector **collects**. It does not classify, score, deduplicate, store or notify. Items
that set a classification field are *rejected* - a source that classifies has produced a
verdict no model computed, which is pattern 3 one layer earlier.

## The part that matters most

`fetchImpl = fetch` is not decoration. **A function that returns its findings is testable
with no database, no network and no API key.** The nine original collectors needed all
three to test at all, which is exactly why none of them were tested. The shared runner now
has 26 offline assertions, including the three failures above.

---

**Reference implementation:** [`core/collector.mjs`](https://github.com/gledach/signals/blob/main/core/collector.mjs)
