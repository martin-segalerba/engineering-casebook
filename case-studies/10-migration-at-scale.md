---
title: "10 · Executing a migration of millions of records"
parent: Case studies
nav_order: 10
---

# Executing a migration of millions of records

**Client:** Civic (US state-level civic advocacy nonprofit)
**Role:** Solutions Engineer: execution, tooling, throttling, data repair
**Stack:** HubSpot imports and custom-coded workflow actions, HubSpot APIs, large-file tooling

[Case study 02](02-crm-data-architecture.md) covers how the migration was designed: match keys,
household modeling, a four-phase import order. This one covers what happened when it ran. The design
was the easier half.

## Context

The plan was to merge a 3.05 million row consumer dataset into a HubSpot portal holding 2.09 million
contacts, then deduplicate the result and append email addresses. Every step exceeded a platform limit.

## Problem

Platform limits are not obstacles you route around once. At this volume they are the shape of the
work:

- **Import file limits.** The source files were a gigabyte each. HubSpot will not accept them.
- **API rate limits.** Deduplication means reading, comparing and merging over a million contacts.
  Done naively, the job dies partway through and leaves the database in a half-merged state.
- **Unique-value collisions.** Email is a unique identifier in HubSpot. Appending emails to 2.5 million
  rows means collisions, and a collision is not an error the importer reports, it is a merge.
- **Spreadsheet tooling corrupts identifiers**, silently, which is the problem in this list that cost
  the most time for the least interesting reason.

## What I did

### Chunking that respects record boundaries

Two one-gigabyte CSVs became six importable files. The uninteresting part is splitting. The part that
matters is that chunk boundaries must fall between records and every chunk needs the header, otherwise
the importer accepts the file and silently misinterprets the first row of each chunk. Chunk counts were
reconciled against the source row count before anything was uploaded.

For the largest dataset the browser-based path could not cope at all, and the upload ran through
tooling built for files this size after several failed attempts through the ordinary route.

### Throttling as a design requirement

Deduplication ran as a custom-coded workflow action against the API, with **throttling built in from
the start rather than added after the first failure**. The job has to survive its own runtime: a merge
pass over a million-plus contacts that dies at 60% leaves a database that is neither the old one nor
the new one, and there is no clean resume from that state.

The merge logic normalizes the address, searches for candidate duplicates, and merges toward the
**oldest record**, which preserves the original creation date and the longest engagement history.
Picking the newest would quietly reset the age of the entire database.

### Running someone else's automation deliberately

Two merge engines ran: my custom workflow, and the platform's own auto-merge beta. Across the main
deduplication pass that produced **56,830 merges: 16,539 from my workflow and 40,291 from the
platform's**, with the platform's output going to a manual review queue rather than being trusted. A
later dataset refresh produced another **327,000 merges**.

Using the vendor's beta was a deliberate choice with a deliberate guard. It was faster and better at
fuzzy matching than anything I would write, and it was also the same class of process that caused the
damage in [case study 01](01-identity-disaggregation.md). The review queue is what made it acceptable:
volume from the beta, judgment from a person, and a record of both.

### The bug that ate identifiers

Identifiers with leading zeros were being corrupted. A spreadsheet opening a CSV infers types, decides
a column of digits is numeric, and strips the leading zeros. The identifier still looks like an
identifier, is the right length minus one, and matches nothing.

Nothing errors. Import succeeds. Match rates come back inexplicably low for one segment of the
population, and the cause is two steps upstream in a tool nobody considered part of the pipeline.

The fix is to keep identifier columns as text at every hop and to never let a spreadsheet be a hop in
a pipeline that carries keys. The detection is to verify identifier length distribution against the
source before importing, which is now a standing check rather than something I do when suspicious.

### Email append and the collision diagnosis

Appending email to the contact base processed 2.5 million rows and left **1.5 million contacts with an
email address**. It also surfaced **750,000 conflicts** caused by unique-value collisions.

Those conflicts are the interesting output. A collision means two records claim the same email, which
means either the data is wrong or the two records are the same person. Diagnosing them as a population,
rather than resolving them row by row, is what turned a stuck import into a finding about the shape of
the data.

## Trade-offs

**Six chunks rather than a streaming integration.** A proper pipeline would have been more elegant and
reusable. This was a migration that runs a handful of times, under a deadline, and chunked imports use
the platform's own validated path. Building infrastructure for a one-time job is its own failure mode.

**Merging toward the oldest record.** Preserves history, and means the surviving record may carry a
stale address that a newer duplicate had right. Property-level precedence, which is
[case study 03](03-property-orchestration.md), is the real answer, and it did not exist yet.

**Accepting the platform's beta output.** More merges than I could achieve alone, against the risk of
merges I did not author. The review queue is what made that acceptable, and I would not have run it
without one.

## Outcome

| Operation | Result |
|---|---|
| Contacts imported to production | 2,091,943 |
| Households imported | 1,261,857 |
| Merges, main deduplication pass | 56,830 (16,539 mine, 40,291 platform beta) |
| Merges, later dataset refresh | ~327,000 |
| Contacts left holding an email address | ~1,500,000 |
| Unique-value conflicts diagnosed | ~750,000 |
| Rows processed for the email append | 2,500,000 |

## How it was verified

Row counts were reconciled at every hop: source rows against chunk totals, chunk totals against import
results, import results against portal counts. A migration that loses forty thousand records loses
them quietly, and only counting at each boundary localizes where.

Identifier integrity was checked by length distribution against the source, which is the check the
leading-zero bug taught me to run before rather than after.

Merge output was sampled and reviewed by a person, with the platform beta's merges queued for review
as a matter of policy rather than spot-checked.

## What I would do differently

The leading-zero corruption was found by investigating a low match rate, which means it was found late
and by luck. A validation step that compares identifier format and length against the source at each
hop would have caught it at the first boundary. That check now exists because of this project, which
is the honest version of how it came to exist.

I would also make the merge pass resumable. It was throttled to survive its runtime, which worked, but
"survives because it is slow enough" is not the same as "can be stopped and restarted." Checkpointing
which records have been processed would turn a job you must not interrupt into one you can.

---

**Related patterns:** [Chunk and throttle against platform limits](../patterns/chunk-and-throttle-against-platform-limits.md) ·
[Spreadsheets corrupt identifiers](../patterns/spreadsheets-corrupt-identifiers.md) ·
[Composite match keys](../patterns/composite-match-keys.md)
