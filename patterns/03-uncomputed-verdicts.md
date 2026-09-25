# 3. Never store an uncomputed verdict

**If the model did not produce the answer, do not write one. Halt instead.**

## The incident

A classifier called an LLM provider that started returning `401`. The error fell through a
generic catch, and the code did what looked reasonable: fell back to keyword matching.

For seven weeks it wrote keyword-guessed classifications into permanent storage. **232 rows
the model never produced.** Nothing errored. The dashboard looked healthy. The rows were
indistinguishable from real ones, and a correlation engine downstream treated them as
evidence - so fabricated inputs became confident conclusions.

## The rule

A failed model is an **outage**, not a degraded mode:

```js
if (isDegraded(classification)) return;   // store nothing
exitOnLlmUnavailable(err);                // exit 2, loudly
```

The trade is explicit and worth stating: with the provider dead, collection now halts and
writes nothing. That is correct. **A halted run announces itself; a run that writes
garbage does not**, and the garbage is permanent in a way the outage never was.

## The generalisation

*Degrading silently is worse than failing loudly whenever the output is stored.* Fallbacks
are fine for a rendered response nobody keeps. They are not fine for a row.

Watch for: a fallback path that produces the same shape as the real path, with no field
recording which one ran. Store the method - `classifyMethod` - so the question is
answerable later.

---

**Reference implementation:** [`docs/decisions/llm-failure-policy.md`](https://github.com/gledach/signals/blob/main/docs/decisions/llm-failure-policy.md)
