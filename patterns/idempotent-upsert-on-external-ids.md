---
title: Idempotent upsert on external IDs
parent: Patterns
---

# Idempotent upsert on external IDs

**Rule.** Every object synced from an external system stores that system's primary key in a dedicated
field that is unique, read-only, and not editable by users. All writes match on that field and upsert.
Re-running any sync must never create a duplicate of anything.

**An editable identifier is not an identifier.** This sounds pedantic until you watch it fail. If the
field can be typed into, it will be typed into, and the uniqueness the whole design depends on is
gone.

**Presence checks are not identity checks.** Logic shaped like "if the record seems to exist, update,
otherwise create" has no reliable way to answer the first half of that sentence without an identifier.
Its practical behavior is to always create.

**Preserve the source hierarchy.** If the source system models parent and child, model them as separate
objects with separate identifiers. Flattening a hierarchy into properties on one record means the child
records overwrite each other, and the overwriting looks like data loss with no obvious cause.

**Order the sync and make it retry-safe.** Fix the execution order so each step only depends on
identifiers written by earlier steps, and make every step safe to run again. Webhooks retry, backfills
happen, and networks fail halfway.

**The failure this prevents:** the entire content of
[case study 06](../case-studies/06-clinical-integration-postmortem.md). No identifier for the parent
entity, identifiers stored as editable text, presence checks instead of matching, and a flattened
model. Every appointment webhook created another deal. Each of those four decisions is individually
recoverable, and together they made the data unusable.
