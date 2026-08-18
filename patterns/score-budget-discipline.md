---
title: Score budget discipline
parent: Patterns
---

# Score budget discipline

**Rule.** In a bounded scoring model, the maximum contributions of all groups must sum to exactly the
ceiling. Not more, not accidentally less.

**Sum higher than the ceiling and the score clamps.** Everyone with reasonable signals hits the
maximum and stops being distinguishable from everyone else with reasonable signals. The model still
produces numbers, and the numbers stop carrying information. This is the state a model tends to drift
into, because every new signal gets added by raising a group limit and nobody subtracts.

**Sum well below the ceiling and the bands lie.** If the reachable maximum is far under the top and
nobody decided that, then thresholds describe score levels almost nobody can reach, and the top band
is empty for structural reasons that look like a finding about the population.

**Headroom is fine when it is deliberate.** Leaving a group's budget unallocated on purpose is
pre-approved room for the next signal, and it means adding that signal later will not require
renegotiating every other group's limit. The failure is unclaimed budget, not unused budget.

**Corollaries worth keeping.**

- One signal, one group. If two groups score the same underlying signal, the model double counts it
  and no limit adjustment fixes that.
- Tier anything repeatable, so volume alone cannot max a group.
- Bands hide magnitude. A record in the lowest band may have a real but low score or literally zero,
  and decisions that turn on "no signal at all" must filter the raw value rather than the band.
- Snapshot the configuration before changing it if the tool keeps no version history. Without a
  snapshot, yesterday's model is unrecoverable, and "compare the distribution before and after" is
  impossible.

**Evidence:** [Case study 05](../case-studies/05-portal-rebuild.md).
