# 5. The coverage block

**Return "I do not know" as structured data. An empty result and a broken collector look
identical, and only one of them is an answer.**

## Why

A human staring at an empty dashboard gets suspicious. **An agent reads `matched: 0` and
reports "nothing happened."** It has no instinct that the silence is wrong, and it will
state the conclusion confidently.

So every tool that reports on collected data returns, alongside the rows:

```json
{
  "matched": 0,
  "coverage": {
    "status": "stale",
    "lastCollectedAt": "2026-08-05T00:00:00Z",
    "trustEmptyResult": false,
    "warnings": ["no data collected for 3 of 4 subjects in scope"]
  }
}
```

`trustEmptyResult` is the field that matters. It is `true` only when the store as a whole
is current **and** every subject in scope is actually being collected. It is `null` when
something matched, so no verdict is needed.

## The detail worth copying

Health is derived from **the newest record, not from the scheduler log**. A hand-driven
deployment is not broken, and reporting it as dead because no cron ran would be a false
alarm that trains people to ignore the field.

## The generalisation

Any tool an agent queries should be able to say *"this is an empty answer"* versus *"I am
not in a position to answer."* Most APIs cannot express the difference, and the agent pays
for it.

---

**Reference implementation:** [`docs/mcp.md`](https://github.com/gledach/signals/blob/main/docs/mcp.md)
