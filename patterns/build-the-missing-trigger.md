---
title: Build the missing trigger
parent: Patterns
---

# Build the missing trigger

**Rule.** Event-driven platforms fire when something happens. Nothing fires when something stops
happening. If you need to know about silence, you have to build the observer yourself, and it has to
live outside the system it watches.

**Why the gap exists.** Automation is built around events: a record changes, a form is submitted, a
call is logged. Absence is not an event. "No calls have been logged since Tuesday" cannot be expressed
as a trigger in a system whose entire vocabulary is things that occurred.

**Why it matters more than it sounds.** Silent failure is the worst failure mode an integration has. A
connector that breaks loudly gets fixed the same day. A connector that quietly stops writing looks
identical, from inside the destination system, to a connector with nothing to write. Data goes missing
in the direction of "less," and less does not announce itself.

**How to build it.**

- **Outside, not inside.** A check that runs inside the monitored system shares its failure modes. If
  the platform is down or the integration's host is broken, the monitor built on that platform is also
  broken, and its silence reads as health.
- **Threshold on a window, not on each event.** You cannot alert on the missing record, because the
  missing record is exactly what you do not have. Alert when a period passes with no activity where
  activity was expected. That turns an absence into something observable.
- **Tune the window to the business.** Short enough that the damage is bounded, long enough that
  normal quiet periods do not fire it. A monitor that cries wolf gets muted, and a muted monitor is
  worse than none because it looks like coverage.
- **Give it an operating procedure and an owner.** Monitoring built by one person and understood by one
  person stops working when that person leaves. Write what to check, what healthy looks like, and what
  to do when it is not, and attach a recurring task so somebody is actually asked.

**Better, when you can afford it:** reconcile counts against the source system rather than watching for
total silence. Window monitoring catches total failure; reconciliation catches partial loss too, which
is the failure that is harder to notice and more corrosive.

**Evidence:** [Case study 11](../case-studies/11-integration-forensics.md), where this was built
because the platform has no such trigger. [Case study 12](../case-studies/12-incident-response.md) is
four incidents that were all detected by a human noticing, which is the argument for why it was worth
building.
