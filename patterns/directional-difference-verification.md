---
title: Directional difference verification
parent: Patterns
---

# Directional difference verification

**Rule.** To prove two sets are the same, build both difference sets and require each to be empty.
Never accept matching counts as proof.

**Why counts fail.** Two segments of 41,203 records each can share not a single member. Count equality
is a necessary condition and a very weak one, and it is exactly the check people run because it takes
five seconds and feels like verification.

**The test.** Build "in A, not in B" and "in B, not in A". Both must be zero. This is one extra
artifact per direction and it converts a plausible claim into a proven one.

**It also localizes the error.** A non-empty difference in one direction only tells you the change was
directional, which is usually a filter that got stricter or looser rather than a filter that got
wrong. Both directions non-empty means the definitions diverge on something structural. The counts
alone tell you neither.

**Generalizes past segments.** Any claim that two representations are equivalent takes the same shape:
a rebuilt query against its original, a migrated table against its source, a refactored function
against the one it replaces. Compare the sets both ways rather than comparing summary statistics.

**Discard the difference artifacts after the check.** They are scaffolding, and leaving them behind
adds to the clutter the rebuild was meant to reduce.

**Evidence:** [Case study 05](../case-studies/05-portal-rebuild.md), where this was the standing
acceptance test for every rebuilt segment.
