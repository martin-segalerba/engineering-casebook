---
title: Patterns
nav_order: 3
has_children: true
---

# Patterns

The reusable part. Each page states a rule, the evidence behind it, and the failure it prevents.
These are written to be read on their own, without any client context.

## Identity and data

| Pattern | The short version |
|---|---|
| [Composite match keys](composite-match-keys.md) | When two systems share no identifier, construct one, store it visibly, and join exactly |
| [Idempotent upsert on external IDs](idempotent-upsert-on-external-ids.md) | An editable identifier is not an identifier |
| [ID reuse is real](id-reuse-is-real.md) | A structurally perfect join can still be semantically wrong |
| [Spreadsheets corrupt identifiers](spreadsheets-corrupt-identifiers.md) | A key that passes through a spreadsheet may stop being that key, silently |

## Changing systems that are already running

| Pattern | The short version |
|---|---|
| [Edit in place versus repoint](edit-in-place-vs-repoint.md) | One edit that 182 references inherit, instead of 182 edits you cannot prove you finished |
| [Map dependencies before changing an enum](map-dependencies-before-changing-an-enum.md) | Removing a value does not remove it from the records that hold it |
| [Expiring state beats boolean flags](expiring-state-beats-boolean-flags.md) | Temporary state should carry its own expiry |
| [Chunk and throttle against platform limits](chunk-and-throttle-against-platform-limits.md) | At scale the limits are the shape of the job, not an obstacle to route around |

## Knowing what your system is doing

| Pattern | The short version |
|---|---|
| [Provenance-first debugging](provenance-first-debugging.md) | Read what wrote the value, do not reason about what should have |
| [Build the missing trigger](build-the-missing-trigger.md) | Nothing fires when something stops happening, so build the observer yourself |
| [Directional difference verification](directional-difference-verification.md) | Equal counts prove nothing about membership |
| [API version trade-offs](api-version-tradeoffs.md) | The current version is not always a superset of the one it replaced |

## Models and scores

| Pattern | The short version |
|---|---|
| [Score budget discipline](score-budget-discipline.md) | Group limits must sum to the ceiling or the score stops distinguishing anyone |
