---
title: "02 · Unifying two large datasets into one CRM schema"
parent: Case studies
nav_order: 2
---

# Unifying two large datasets into one CRM schema

**Client:** Civic (US state-level civic advocacy nonprofit)
**Role:** Solutions Engineer: data architecture, match logic, import sequencing
**Stack:** HubSpot CRM (contacts and a custom household object), voter file, consumer data file, calculated properties

## Context

Civic held roughly 2.09 million contacts in HubSpot, built from a voter file and web and canvassing
activity. The organization licensed a second, larger consumer dataset of 3.05 million records with 167
columns, and needed it merged into the same CRM: enriching the people already there, adding the people
who were not, and modeling households as first-class records rather than as an address string repeated
across contacts.

The two datasets have no shared primary key.

## Problem

Three questions had to be answered before a single row could be imported.

**Which records in the two systems are the same person,** given no common identifier.

**What is a household,** and how does a contact get attached to the right one when the two sources
disagree about addresses.

**In what order does any of this get imported,** such that a failure partway through leaves the portal
in a recoverable state rather than a worse one.

## Constraints

- **Imports at this scale are not casually reversible.** Two million rows written wrongly is not
  something to undo by hand.
- **HubSpot dedupes contacts on email at import,** so anything producing two rows with one email
  silently merges the people it was supposed to keep apart.
- **A merge process was already running against the portal** and had already caused damage, which is
  the subject of [case study 01](01-identity-disaggregation.md). Import design had to avoid feeding it.

## What I designed

### Match keys: identity when there is no shared identifier

A match key is a calculated property that concatenates identifying values into a single string, so two
records from different systems can be compared with an exact match rather than fuzzy logic.

```
Contact match key   =  FIRSTNAME | LASTNAME | BIRTHDATE | ZIP | SEX
Household match key =  ADDRESS | ZIP
```

Building them as calculated properties inside HubSpot, and computing the identical string in the
source data, means the join runs in the import tool rather than in a separate matching pass. It is
also inspectable: anyone can open a contact and read the key that matched it.

Matching runs in precedence order. State voter ID first where present, because a real identifier beats
a constructed one. The composite key only as fallback. Households have no shared identifier between
sources at all, so they always match on the constructed key.

### Rules of thumb, written down

The team needed rules they could apply without me. Five, in plain language:

1. **The voter ID is the primary identifier** whenever present, and it takes precedence over any
   composite key.
2. **One household number, one household.** Contacts sharing a household number belong to the same
   household record. This is definitive when the number exists.
3. **Count the people.** Where the source states how many people a household has, use it to verify
   integrity. Every household should have exactly that many contacts attached.
4. **Housing attributes live on the household**, not on the contact, even when they also exist on
   contacts for matching purposes.
5. **Match from the contact, trust the household.** Use contact-level data to find or create the
   household, then treat the household as the system of record for shared values like location.

Rule 4 and 5 together are what stop a household from being an unowned copy of whichever contact was
imported last.

### Import order designed around recoverability

Four phases, sequenced so that each one only depends on identifiers established by the previous one:

**Phase 1: voter ID contacts and households.** Generate match keys on both sides. Import and update
only contacts that carry a voter ID, assigning household numbers. Associate to existing households by
match key. Then create the households that do not yet exist. No deduplication at this stage: it is
deferred deliberately, because deduplicating before all identifiers are populated is how you merge two
people who were about to become distinguishable.

**Phase 2: contacts without a voter ID.** Import by composite match key, updating existing records
only, and assign household numbers. Then associate contacts to households by that number. Contacts
matching neither a voter ID nor a match key are imported last, as orphans, flagged for enrichment or
manual review rather than guessed at.

**Phase 3: deduplication.** Only now, with both household numbers and match keys populated on
everything, does the dedup process run against both signals.

**Phase 4: full enrichment.** Export the record identifiers from both systems, rejoin them, then bulk
import the remaining 160-odd columns against HubSpot's own record ID, so property updates land on the
correct profile by primary key rather than being re-matched.

The ordering principle is that every phase writes only identifiers and associations that later phases
depend on, and the expensive, wide, hard-to-reverse write happens last, when matching is no longer
part of the operation.

## Trade-offs

**Deduplication deliberately deferred to phase 3.** The instinct is to dedupe early so later imports
hit clean data. That is backwards here: the identifiers that make dedup safe do not exist until phases
1 and 2 have run. Deduping first means merging on soft signals, which is exactly the process that
caused the damage in case study 01.

**Match key stored as a property rather than computed at import time.** It costs a property and it
must be regenerated when its inputs change. It buys inspectability, which matters when a non-technical
team has to trust and debug the matching.

**Orphans kept rather than dropped or force-matched.** Contacts matching nothing are imported and
flagged. Force-matching them on weaker criteria would raise the coverage number and lower the truth of
the database.

## Outcome

Measured against the live data before import:

| Measure | Value |
|---|---|
| Contacts in HubSpot | 2,091,943 |
| Carrying a state voter ID | 1,972,442 |
| Match rate between the two datasets on voter ID | 71% |
| Contacts identifiable in a single keyed import | ~1,400,000 (roughly 67% of all contacts) |
| Households in HubSpot before | 1,261,857 |
| Households in the consumer dataset | 2,065,558 |
| Household match rate | 51% |
| Projected households after | ~2,270,000 |

Knowing these numbers before importing is the point of the exercise. They set expectations with the
client, they sized the orphan population that would need manual attention, and they made it possible
to say what the import would and would not accomplish before anyone committed to it.

## How it was verified

Match rates were measured by running the join against the real datasets before any import, rather than
estimated. Household integrity has a built-in check from rule 3: where the source states a household's
population, the count of attached contacts must equal it, which catches both over-attachment and
missed associations without needing a separate audit.

The most valuable verification was negative, and it is documented in
[case study 01](01-identity-disaggregation.md): the crosswalk that appeared to link the two datasets
one-to-one was tested against name, address and birth year, and roughly a third of the sample
disagreed. Match rate as a headline number would have hidden that entirely.

## What I would do differently

The composite contact key includes date of birth and ZIP, both of which are missing or wrong on part of
the population, and a null in any component silently produces a key that cannot match anything. I would
build the key with explicit handling for missing components, and report key completeness as a
distribution up front, so the population that structurally cannot match is visible from the start
rather than appearing as an unexplained orphan count at the end.

---

**Related patterns:** [Composite match keys](../patterns/composite-match-keys.md) ·
[ID reuse is real](../patterns/id-reuse-is-real.md)
