---
title: Chunk and throttle against platform limits
parent: Patterns
---

# Chunk and throttle against platform limits

**Rule.** At scale, platform limits are not obstacles to route around. They are the shape of the job,
and they belong in the design from the first version rather than being added after the first failure.

**The two limits that matter.** File size and row count on imports, and API rate limits on anything
that reads or writes record by record. Both are hit at volumes that ordinary work never approaches,
so neither is in anyone's mental model until the job dies.

**Chunking: the boundary is the whole problem.** Splitting a large file is trivial; splitting it
correctly is not.

- Chunk boundaries must fall **between records**, never inside one.
- Every chunk needs the **header row**, or the importer accepts the file and misreads the first record
  of each chunk. This does not error.
- **Reconcile counts** at every hop: source rows against chunk totals, chunk totals against import
  results, import results against destination counts. A job that loses forty thousand records loses
  them quietly, and counting at each boundary is what localizes where.

**Throttling: the job must survive its own runtime.** A pass over a million records that dies at 60%
leaves a database that is neither the old state nor the new one, and there is usually no clean resume
from there. Throttle so the job completes, even though that makes it slower than the platform would
briefly allow.

**Better than throttling: checkpointing.** "Survives because it is slow enough" is not the same as
"can be stopped and restarted." Recording which records have been processed converts a job you must
not interrupt into one you can, and it is the difference between a bad afternoon and a bad week.

**Direction matters on merges.** When deduplicating, decide deliberately which record survives.
Merging toward the oldest preserves creation dates and engagement history. Merging toward the newest
silently resets the age of the entire database, and that is not recoverable.

**Evidence:** [Case study 10](../case-studies/10-migration-at-scale.md). Two one-gigabyte files into
six importable chunks, a throttled merge pass producing 56,830 merges, and 2.5 million rows processed
for an email append.
