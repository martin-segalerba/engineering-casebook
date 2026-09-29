---
title: Map dependencies before changing an enum
parent: Patterns
---

# Map dependencies before changing an enum

**Rule.** Before changing or removing a value from an enumeration that a live system branches on,
enumerate every place that reads it. The enumeration is the work. The change itself is usually
trivial.

**What counts as a load-bearing enum.** Lifecycle stages, pipeline stages, status fields, category
pickers, any field whose values appear in filter conditions rather than being merely displayed. These
accumulate readers invisibly: every segment, automation condition, report and integration mapping that
mentions the value is a dependency, and none of them announce themselves.

**Why it goes wrong.** Removing an enum value does not remove it from the records that hold it. The
stored value stays and the filters stop matching, so a population silently falls out of every segment
that referenced it. Nothing errors. A group of records simply stops being addressed, and that is
discovered weeks later by someone asking why a number dropped.

**The procedure.**

1. **Enumerate the readers.** Every segment, automation, report and integration mapping. Use the
   platform's own dependency view where it exists, and do not trust it to be complete.
2. **Count before you design.** One redesign meant mapping 46 automations and 90 segments. The count
   is what converts "let's clean up lifecycle stages" into a scoped piece of work with a cutover plan.
3. **Sequence the cutover.** New values in place and populated first, readers repointed second, old
   values removed last. Each step is individually reversible; the combined change is not.
4. **Expect one that cannot be deleted.** Platforms protect certain system values, and discovering
   that mid-cutover is worse than discovering it while planning.
5. **Verify by population, not by configuration.** After the change, the count of records in each
   state should reconcile against the count before. Configuration that looks right and a population
   that moved are different things.

**Related:** this is the same hazard as
[edit in place versus repoint](edit-in-place-vs-repoint.md), one level down. That pattern is about a
definition referenced from many places; this one is about a *value* referenced from many places. Both
resolve to: count the references before you touch the thing.

**Evidence:** [Case study 13](../case-studies/13-scoring-models.md) and
[case study 05](../case-studies/05-portal-rebuild.md). Four lifecycle redesigns across four
organizations, where the mapping consistently took longer than the change.
