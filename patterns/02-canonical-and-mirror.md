# 2. Canonical and mirror

**The database is the source of truth. Files on disk are a mirror. Every writer
read-modify-writes the record and then mirrors - nobody writes the file directly.**

## The incident

Generated documents in the reference implementation carry two kinds of content in one
file: an AUTO section a model regenerates, and a HUMAN section holding the most valuable
material in the system - notes a person wrote after a real conversation.

The obvious migration - "make the database canonical, sync it to disk" - **would have
destroyed every one of those human-written sections the first time the scheduler
regenerated a document**, because the database had never seen them. They were only ever
written to disk.

## The rule

```
EVERY WRITER READ-MODIFY-WRITES THE STORED RECORD AND MIRRORS TO DISK.
NOBODY WRITES THE FILE DIRECTLY.
```

Reads prefer the database and fall back to disk, so a deployment mid-migration keeps
working and a fresh clone with files but no rows still renders.

The read-modify-write is not ceremony. Regeneration can take ten minutes; a human edit
landing during that window must survive, and it only survives if the writer re-reads
immediately before writing instead of holding a stale copy.

## What it caught later

The same layer had a second bug for months: transcripts were written **only** to disk, in a
directory excluded from both version control and the deploy. Every redeploy erased the
archive, an API endpoint that read it could never return anything, and the "already have
it" check was a file-existence test - so the collector re-fetched every item, for ever.
Three silent failures from one missing database write.

## Portable?

**Pattern only.** But the read-modify-write shape transfers verbatim to anything that is
part generated and part human-edited.

---

**Reference implementation:** [`core/artifacts.mjs`](https://github.com/gledach/signals/blob/main/core/artifacts.mjs)
