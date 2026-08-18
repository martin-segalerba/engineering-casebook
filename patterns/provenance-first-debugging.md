---
title: Provenance-first debugging
parent: Patterns
---

# Provenance-first debugging

**Rule.** When a value is wrong, do not reason about what should have written it. Read what did. The
unit of truth is source plus identifier plus timestamp, and any platform that records those is telling
you the answer directly.

**Why the intuitive approach fails.** Reasoning from the automation is reasoning about intent. In a
system with hundreds of automations, several imports and a bidirectional sync, intent and behavior
diverge constantly, and the automation that looks responsible is usually not the one that wrote the
value.

**Group by write event, not by record state.** A record's current state is the accumulated result of
many writes from many sources, so it does not describe any one of them. Changes sharing a source
identifier and a timestamp are one write event, and a single write event is internally consistent:
those values genuinely belong together and genuinely came from one place.

**Never read a merge snapshot as a person.** When a platform writes a post-merge snapshot, that
snapshot is the survivor's blended state: one field from one person, another field from another.
Reading it as though it described a coherent record produces confident, plausible, wrong output. Use
the delta instead, meaning only the values that actually changed at that moment. Values that did not
change are the survivor's own data echoing.

**Order the investigation cheapest first.** Classification, then exclusion state, then membership,
then automation history, then the value's own history. Each step is more work than the last, and one of
the first three answers the question most of the time.

**Design for it.** If you are building the system, emit source, identifier and timestamp on every
write, and surface them where a human can read them. The reconstruction engine in
[case study 01](../case-studies/01-identity-disaggregation.md) shows the source of every single value
in its review UI, which is what made its output auditable rather than merely plausible.
