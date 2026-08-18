---
title: Expiring state beats boolean flags
parent: Patterns
---

# Expiring state beats boolean flags

**Rule.** When state is temporary, express it as something carrying its own expiry rather than as a
boolean that something else has to remember to clear.

**The anti-pattern.** A shared boolean flag, written by many automations and cleared by many others.
It fails in both directions. If the automation that clears it never runs, the record stays in that
state forever. If a different automation clears it early, the state ends mid-series. Nothing owns the
flag, so everything fights over it, and no single place explains the current value.

**The pattern.** Give the state an expiry built in. A dated record, a timestamp field with a deadline,
a TTL. Whatever reads the state evaluates the expiry as part of the read.

In the implementation this comes from, temporary email suppression became a dated task associated to
the contact, and the suppression segment is a filter for contacts with an unexpired task of that type.
Nothing has to turn suppression off. When the due date passes, the filter stops matching.

**What it buys beyond correctness:**

- **Exceptions become data edits, not logic edits.** Releasing someone early is completing their task.
  Holding them longer is changing a date. Neither requires touching automation, so a non-technical
  operator can handle day-to-day exceptions safely.
- **The state is self-documenting.** Each record names what created it and when it ends, so "why is
  this in that state" is answered by reading the records rather than by tracing automation.
- **Precision comes free.** A timestamp expresses a four-and-a-half day hold as naturally as a
  four-day one.
- **Concurrency stops being a problem.** Several overlapping holds coexist and the latest expiry wins,
  where a boolean can only be set or unset and the second writer destroys the first one's intent.

**Cost.** One extra level of indirection, and the state now lives across many records rather than one
field, so bulk reasoning about it needs a query rather than a glance.

**Evidence:** [Case study 05](../case-studies/05-portal-rebuild.md).
