# 4. Read-only agent surface

**An agent reads by default. Every paid or state-changing action is opt-in, per
deployment, behind a shared spend ceiling.**

## Why the default matters

An agent exploring a new tool **calls everything once to see what it does.** That is
reasonable behaviour and you should design for it. Someone who clones your repo and wires
it into their assistant has not agreed to let that assistant spend their credit.

```js
export const policy = {
  allowActions: [],              // empty = purely read-only
  budget: { dailyUsd: 2.00, perCallUsd: 0.30 },
};
```

Enabling an action should be a sentence someone writes on purpose, in a gitignored local
file, not a default they inherit.

## The ceiling is shared, and that is the point

One rolling 24-hour number drawn down by the scheduler, the CLI **and** every agent. An
agent cannot spend the budget the scheduled pipeline still needs. Sizing it from observed
spend rather than instinct means the number survives review.

Two refinements that came from failures:

- **Fail closed.** If the spend ledger cannot be read, refuse. An unenforceable ceiling on
  a paid path is worse than no ceiling, because it reads as protection.
- **Refuse before spending.** A pre-flight estimate that declines a run beats discovering
  the overrun in the bill.

---

**Reference implementation:** [`config/agent-policy.default.mjs`](https://github.com/gledach/signals/blob/main/config/agent-policy.default.mjs)
