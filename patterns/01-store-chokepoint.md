# 1. Store chokepoint

**One module talks to the database. Everything else talks to that module. A test enforces it.**

```js
// core/store.mjs is the ONLY place that opens a client.
export async appendSignal(), loadAllSignals(), alreadySeen(), seenHashIds(), updateSignal()
```

## Why

This is the pattern everyone nods at and half of them skip, because the cost only shows up
later. A schema change with N callers has N places to break, and you find the Nth in
production. With a chokepoint it has one.

The second benefit is the one nobody predicts: **the chokepoint is where you discover the
query you should have written.** The reference implementation ran `alreadySeen(hashId)`
once per item for months. Most of a collection run is duplicates, so most of a run was
round-trip latency spent learning "seen it". Adding a batch `seenHashIds()` was a 25-line
change *because there was one place to add it*.

## The rule that makes it hold

Not "please use store.mjs". An assertion:

> `core/store.mjs` is the only module importing the database client, and the smoke suite
> fails if another one does.

A convention decays at the first deadline. An assertion does not.

## Portable?

**The pattern, not the code.** The reference implementation is 823 lines with 59 references
to its own domain. Copy the boundary, write your own module.

---

**Reference implementation:** [`core/store.mjs`](https://github.com/gledach/signals/blob/main/core/store.mjs)
