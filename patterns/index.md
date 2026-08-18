---
title: Patterns
nav_order: 3
has_children: true
---

# Patterns

The reusable part. Each page states a rule, the evidence behind it, and the failure it prevents.
These are written to be read on their own, without any client context.

| Pattern | The short version |
|---|---|
| [Composite match keys](composite-match-keys.md) | When two systems share no identifier, construct one, store it visibly, and join exactly |
| [Idempotent upsert on external IDs](idempotent-upsert-on-external-ids.md) | An editable identifier is not an identifier |
| [ID reuse is real](id-reuse-is-real.md) | A structurally perfect join can still be semantically wrong |
| [Expiring state beats boolean flags](expiring-state-beats-boolean-flags.md) | Temporary state should carry its own expiry |
| [Edit in place versus repoint](edit-in-place-vs-repoint.md) | One edit that 182 references inherit, instead of 182 edits you cannot prove you finished |
| [Directional difference verification](directional-difference-verification.md) | Equal counts prove nothing about membership |
| [Score budget discipline](score-budget-discipline.md) | Group limits must sum to the ceiling or the score stops distinguishing anyone |
| [Provenance-first debugging](provenance-first-debugging.md) | Read what wrote the value, do not reason about what should have |
| [API version trade-offs](api-version-tradeoffs.md) | The current version is not always a superset of the one it replaced |
