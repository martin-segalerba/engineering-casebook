---
title: "01 · Identity disaggregation at two-million-contact scale"
parent: Case studies
nav_order: 1
---

# Identity disaggregation at two-million-contact scale

**Client:** Civic (US state-level civic advocacy nonprofit)
**Role:** Solutions Engineer, sole designer and builder
**Stack:** Python, FastAPI, SQLite, HubSpot v3 and v1 APIs, vanilla JS with Server-Sent Events

## Context

Civic runs voter mobilization and economic-justice campaigns on a HubSpot portal holding roughly 2.09
million contacts, assembled over years from a voter file, a consumer data file, canvassing software
and web forms.

An automated deduplication process had been running against that portal. It matched on soft signals,
mostly name and address, and it merged. HubSpot merges are one-directional: the loser record is
absorbed into the winner and the loser ceases to exist as a record. There is no supported un-merge for
this at volume.

The result was super-contacts. Single HubSpot records that were not one person with duplicate entries,
but hundreds of genuinely different people fused into one row. The worst example held over a thousand
absorbed identities.

## Problem

Everything downstream was wrong, quietly. A super-contact carries one email address, so hundreds of
people were unreachable. It carries one address, one giving history, one engagement record, so
segmentation, targeting and reporting were all computed against a fiction. Nobody could say how many
supporters the organization actually had.

The task was to reconstruct the original people from what HubSpot still remembered, at a fidelity high
enough to re-import them as real records.

## Constraints

- **The merges could not be reversed through the platform.** Whatever was recoverable had to come out
  of history and audit surfaces.
- **The output had to be importable.** A report describing the damage was not useful. The deliverable
  had to be CSVs HubSpot would accept.
- **Splitting wrongly is worse than not splitting.** Fragmenting one real person into six records, or
  attaching someone's email to a stranger, creates damage that is harder to detect than the original
  merge. This constraint drove the whole design.
- **Reads only.** No write scope, no destructive operation against the live portal.

## What I designed

A local reconstruction engine. It reads a set of contact IDs or a HubSpot list, pulls everything the
platform still knows about each super-contact, resolves the identity fragments against authoritative
external datasets held locally, and streams reconstructed people into a review UI while writing two
CSVs: the record IDs to delete, and one row per reconstructed person ready to import.

```
contacts / segment
   -> v3 batch read (property history + sourceType)
   -> v1 profile (absorbed VIDs, merge audits, per-value source VIDs)
   -> local dataset resolution (SQLite)
   -> identity extraction -> source attribution -> resolve and unify
   -> confidence tiers -> two CSVs
```

### Merges are recoverable from history

`hs_merged_object_ids` is a cumulative, semicolon-joined list of every absorbed record ID. Diffing
consecutive versions of that property yields the individual merge events and their timestamps. That
turns "this record was merged at some point" into an ordered log of exactly which record joined when,
with no extra API surface required.

### The newer API exposes less history than the one it replaced

The v3 API caps property history at 45 versions. A contact merged three hundred times has silently
lost its oldest identities there. This is the constraint that decides whether the project is possible
at all, because the oldest identities are the ones nobody else can recover.

The legacy v1 profile endpoint is uncapped on the payload that matters. On a contact where v3 exposed
44 identities, v1 returned 1,001. Three payloads carried the reconstruction:

| Payload | What it gives | Cap |
|---|---|---|
| `identity-profiles` | One entry per absorbed record, each with its own email and GUIDs | Uncapped |
| `merge-audits` | One record per merge: loser ID, timestamp, loser name, properties moved | ~500 |
| `source-vids` per property version | The chain of records a value passed through; index 0 is the record that originated it | Follows history |

Grouping every property's versions by originating record ID reconstructs a per-loser property
snapshot. That is the core move of the whole engine.

### Never trust the merge snapshot

HubSpot writes a `MERGE_OBJECTS` snapshot at each merge. It is the winner's post-merge state, which is
a Frankenstein: address from one person, city from another. Reading it as though it described a person
produces confident, plausible, wrong records.

Attribution instead groups changes by `sourceId` plus timestamp, which is one write event and
therefore one internally consistent person, and uses the merge **delta**, meaning only the values that
actually changed at the merge timestamp. Values that did not change are the winner's own data echoing,
and attributing them to the loser is how a reconstruction quietly invents people.

### Anchor on identifiers that are provably never reused

Reconstruction resolves against two locally indexed datasets: a 2,341,234-row voter file and a
3,048,448-row consumer file, plus two crosswalks. Not all identifiers are equally safe:

| Key | Coverage | Quality |
|---|---|---|
| Consumer file individual ID | HubSpot, consumer file | Unique across all 3,048,448 rows, zero reuse. The best key. |
| State voter ID | All three sources, via crosswalk | The only key spanning everything, and roughly 3.6% get reassigned to a different person over time |
| Voter file internal ID | HubSpot, voter file | Exact and direct, but does not reach the consumer file |
| Email | HubSpot, consumer file via import crosswalk | Can be shared by several people |

Where the datasets and HubSpot disagreed on a name, address or date of birth, the datasets win.
HubSpot's fields are exactly the ones a merge may have overwritten. HubSpot contributes the
identifiers and the engagement, and the clean sources contribute the person.

### The rules that stop over-splitting and over-collapsing

- Two records are the same person only if they share a hard identifier. Household ID and address
  separate people, and never unify them.
- Every bridge between the two datasets built on state voter ID is confirmed against birth year, or
  last name where birth year is missing. A mismatch means the ID was reassigned, and the bridge is
  rejected.
- An exported person must carry both a name and a hard identifier. Fragments with neither are dropped
  rather than guessed at.
- A shared email is assigned to the earliest owner, meaning whoever held it first in HubSpot. Everyone
  else keeps a review flag, and the email is never duplicated in the output, because HubSpot dedupes
  on email at import and a duplicate would re-merge the people I just separated.
- Every person carries all of their hard identifiers, so the global dedup pass unifies someone who was
  merged into two different super-contacts.

## Trade-offs

**Integration platform versus code.** The first approach was built as a workflow in an integration
platform. It was abandoned. The reconstruction is recursive, needs a local index over six million rows
to be usable, and required rapid iteration on identity rules that were changing daily as the data
revealed itself. A scripted engine with local datasets was the better fit, and the platform work was
written off rather than defended.

**Write-back versus CSV.** The engine performs zero writes. Output is two CSVs a human reviews before
importing. For an operation that deletes records and creates thousands of new ones, a reviewable
artifact and an explicit human gate were worth more than the convenience of writing directly.

**Confidence tiers instead of a single threshold.** Every reconstructed person is labeled by how it
was identified: confirmed by a dataset ID, recovered from HubSpot data via the legacy API, or
identified by a real email only. The reviewer decides what to accept per tier, rather than accepting
one global cut chosen by me.

## Outcome

Reference run over a 191 super-contact test segment:

| Metric | Result |
|---|---|
| People reconstructed | 9,916 |
| Carrying both a name and a hard ID | 100% |
| Over-split (one person fractured into several) | 0 |
| Anchorless rows (no reconstructable identity) | 0 |
| Import collisions (two rows sharing an email) | 0 |
| Misattribution (a value on the wrong person) | ~0 |

Per super-contact cost is roughly two v3 calls plus one v1 call. Dataset lookups are local and
instant, and write-back costs nothing because the output is a CSV. The cost per contact does not
change with volume, so the approach scales linearly to the full target population.

## How it was verified

I defined the error taxonomy before trusting any output, and split it into two families.

Family 1 is the damage already present in HubSpot that the engine inherits and reverses: name-collision
merges, stepover contamination from imports overwriting fields without a formal merge, Frankenstein
records, shared identifiers, and history truncation at the 45-version cap.

Family 2 is the risk created by the cure, and this is the family that was audited to zero:
over-collapse, over-split, misattribution, anchorless rows, and import collisions. Each class got a
diagnostic over the full test segment, and each failure fed a rule change followed by a re-run.

Two findings came out of that loop and changed the design:

**A planned merge of ~698 "same person" pairs was rejected.** The pairs came from a crosswalk between
the two datasets, joined on state voter ID. Checking them rather than trusting them: only 64% agreed
on name and address, 30% had different addresses, and 5% were entirely different people. The crosswalk
table itself is a perfect one-to-one bijection, which is why it looked trustworthy. The failure is
that state voter IDs get reassigned to different people over time. A structurally perfect join can
still be semantically wrong, and the only way to find out is to test it against an attribute the join
does not use.

**One reconstruction absorbed 116 distinct people into a single false person.** The merge audit field
naming the merged-from email holds the *winner's* primary email, not the loser's. Using it to identify
losers collapsed 116 different people onto one email. Fixed by sourcing loser emails only from the
per-record identity profiles.

## What I would do differently

The two crosswalks were built ad hoc during development rather than folded into the standard indexing
step, so reproducing the environment on another machine takes manual work that is not self-documenting.
That is a productionization gap I would close before handing the tool to anyone else.

I would also add a held-out validation set: a group of super-contacts whose true composition is
established independently, scored automatically on every run. The audit loop was thorough but manual,
and manual audits stop being run.

The larger lesson is about sequencing. This entire project exists because an automated dedup process
was allowed to merge on soft keys without a reversibility plan. Merging is destructive and cheap to
start, and reconstruction is expensive and only partially possible. Any dedup that matches on anything
softer than a hard identifier should be reviewed before it writes, or should associate rather than
merge, which is exactly the pattern adopted in [case study 05](05-portal-rebuild.md).

---

**Related patterns:** [Provenance-first debugging](../patterns/provenance-first-debugging.md) ·
[ID reuse is real](../patterns/id-reuse-is-real.md) ·
[API version trade-offs](../patterns/api-version-tradeoffs.md)
