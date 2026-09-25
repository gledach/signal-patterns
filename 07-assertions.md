# 7. Assertions as guardrails

**Every rule in this repo is held by a test. A rule that is only documented is a comment.**

## Why this one is last and matters most

The other seven patterns all erode. Someone adds a direct database call because they are
in a hurry. Someone puts a per-subject job on the wrong schedule. Someone writes a fallback
that stores a guess. None of it is malice and none of it is visible in review - each change
is one line, in a plausible place.

So each pattern gets an assertion that fails when it is violated:

| Rule | What fails |
|---|---|
| One database gateway | another module imports the client |
| Collectors do not write | a collector calls a store function |
| No uncomputed verdicts | a writer drops its degraded-verdict check |
| Per-subject jobs off the hot schedule | a fan-out job appears in the frequent tier |
| No private paths in tracked files | a generated block reintroduces one |

## Two things learned the hard way

**Assert on behaviour, not on source text, where you can.** A security control checked only
by a regex over its own file is one nobody has executed. Extract it into a module that can
be imported and called - even if that costs a file.

**Never allowlist the files you thought of.** One check here named four files and passed,
while nineteen others were violating the same rule. Scan everything and allowlist the
genuine exceptions, which should be few enough to name.

## The test that pays for itself

The single highest-value assertion in the reference implementation checks that a fresh
clone, with no account and no API key, produces a working system. Every piece of that works
individually; it breaks as a whole, silently, and it is the first thing a stranger does.

---

**Reference implementation:** [`test/smoke.mjs`](https://github.com/gledach/signals/blob/main/test/smoke.mjs)
