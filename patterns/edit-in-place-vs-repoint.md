---
title: Edit in place versus repoint
parent: Patterns
---

# Edit in place versus repoint

**Rule.** When a definition is referenced from many places and you want to change it, edit the original
in place. Do not build a replacement and repoint the references to it.

**The arithmetic.** One list in a CRM portal was referenced in 182 places: send exclusions, workflow
conditions, nested list definitions. Repointing means finding and editing all 182, and any reference
you miss keeps silently enforcing the old definition. There is no way to prove you found them all.

Editing the original keeps its ID stable. All 182 references keep working untouched, and every one of
them inherits the new definition at the same instant. One edit instead of 182, atomically.

**Build the replacement anyway.** The rebuilt version is still worth building, as a reference to copy
from and to compare against. It just is not what goes into service. Adoption is copying its definition
onto the original, filter by filter, then deleting the now-redundant copy.

**The hazard, and its procedure.** Editing a definition in place makes membership churn for a moment.
Anything using that definition as a trigger with re-enrollment enabled will fire on that churn. So:
check the dependency panel first, pause those consumers, edit, verify, un-pause. The dependency check
is not optional, and it is the step people skip.

**Verify by difference, not by count.** See
[directional difference verification](directional-difference-verification.md). Equal sizes prove
nothing about membership.

**When repointing is right.** When the old definition must remain available under its own identity, or
when you genuinely want references migrated one at a time so each consumer is reviewed. Both are real,
and both should be a decision rather than the default.

**Evidence:** [Case study 05](../case-studies/05-portal-rebuild.md).
