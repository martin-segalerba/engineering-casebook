---
title: "13 · Three scoring models in production"
parent: Case studies
nav_order: 13
---

# Three scoring models in production

**Clients:** Relief, Signal, Oberon (anonymized)
**Role:** Solutions Engineer: model design, workshops, build, deployment, iteration
**Stack:** HubSpot scoring and segmentation, Salesforce-synced giving and revenue data

Scoring appears as one section of [case study 05](05-portal-rebuild.md). This is the practice across
three organizations, because the decisions that recur are more useful than any single model.

## Why organizations ask for this

The request is always some version of "tell us who to contact." Underneath it is a team rebuilding
audiences by hand for every send, where targeting quality depends on who happened to build it and no
two people build it the same way.

A scoring model replaces per-send handiwork with standing segments that re-evaluate themselves. The
engineering is in deciding what the numbers mean and what happens when they are wrong.

## The decision that shapes every model: one axis or two

A single score collapses two unrelated questions into one number: **how engaged is this person** and
**how valuable are they**. Those move independently, and a contact who scores 50 might be highly
engaged and worth little, or disengaged and worth a great deal. A single number cannot tell you which,
and it is the difference between two completely different actions.

All three models are two-axis for that reason, though the axes are named for their domain.

**Relief, a nonprofit: Engagement and Fit.** Fit is built on RFM principles from giving history:
recency, frequency and depth of giving, with decay so a lapsed major donor does not read as current,
and penalties for signals that should subtract. Suggested ask amount and acquisition channel
contribute. Engagement is behavioral: opens, clicks, visits.

**Signal, B2B SaaS: a dual-brand model.** Two brands share a portal and a contact database, so a
single model would average two different definitions of a good prospect into something true of
neither. The model scores each brand separately. Interactions are split into **soft and hard signals**
with decay, and an engagement stage workflow turns the score into a state the rest of the system can
act on.

**Oberon, B2B SaaS: scoring alongside lifecycle definition.** The scoring work here was inseparable
from defining what the lifecycle stages meant in the first place, because a model that feeds stages
nobody has defined produces precise numbers about an undefined thing.

## What carries across all three

**The budget has to sum to the ceiling.** If group limits total more than the maximum, everyone with
reasonable signals pins at the top and the score stops distinguishing anybody. If they total far less
and nobody decided that, the upper bands describe scores almost nobody can reach. This is the single
most common defect in an inherited model, and it arrives by accretion: every new signal gets added by
raising a limit, and nobody ever subtracts. It has its own pattern,
[score budget discipline](../patterns/score-budget-discipline.md).

**One signal, one group.** If two groups score the same underlying behavior, the model double counts
it and no amount of limit tuning fixes that.

**Decay for behavior, never for identity.** Engagement should fade, because a click two years ago is
not evidence about today. Fit attributes that are facts about a person should not decay on a timer.
Getting this backwards produces a model that slowly forgets who its best supporters are.

**Bands hide magnitude.** A contact in the lowest band may have a real but low score, or literally
zero. Those are different situations, and decisions that turn on "no signal at all," like
deliverability gates, must filter the raw number rather than the band.

**Where the score does not decide.** In the nonprofit model, major donors do not convert by email, so
their scores inform a relationship manager rather than mass-send inclusion. Writing down where a model
should be ignored is part of shipping it. Every scoring model gets over-applied by someone who was not
in the workshops, and the countermeasure is documentation rather than more tuning.

**Snapshot before changing anything.** The scoring tool keeps no version history. Without a dated
snapshot of every group, criterion, tier and limit, yesterday's model is unrecoverable and "compare
the distribution before and after" is impossible.

## Two things that were not technical

**A platform bug forced a rebuild.** During the nonprofit build, a product defect in the scoring tool
required escalation to the vendor and a rebuild of work already done. The lesson is about sequencing:
the snapshot discipline above is what made rebuilding possible rather than starting over, and it was
in place because of the general rule, not because I expected a bug.

**A client AI policy, navigated rather than bypassed.** Threshold setting is empirical. You need to
look at real data from accounts that converted and see where the bands should fall. The client had a
strict policy on AI tooling with data.

The path of least resistance is to do the analysis quietly. Instead: proposed the vendor's official
plugin through the proper channel, and when that was not acceptable, pivoted to working from anonymized
extracts, which satisfied the policy and still answered the question. Approved without needing to
escalate to their IT organization.

I include it because it is a real constraint on technical work and there is a right way through it.
The wrong way produces better thresholds and a client who finds out later how you got them.

## Outcome

- Three models in production across three organizations, including one deployed to a supporter base of
  roughly 1.84 million contacts.
- The dual-brand model went live in February with QA on the resulting distribution, which is the check
  that catches clamping before anyone acts on the output.
- Audience construction moved from per-send manual work to standing segments that re-evaluate as
  engagement decays, with no manual pruning.
- Each model shipped with a walkthrough for the team that operates it, and iterated on their feedback
  after launch rather than being handed over as finished.

## How it was verified

**Distribution before belief.** Preview the distribution and check the shape before trusting any band.
A model where everyone clusters at a ceiling or a floor is broken regardless of how sensible the rules
read.

**Cross-count against the existing segmentation plan.** Count how many contacts from each existing
segment land in each band. The pairing between segments and bands is an expectation, not an enforced
rule, so it has to be re-derived rather than assumed, and where it disagrees, one of the two is wrong
and it is worth knowing which.

**Spot-check named records.** Test against two or three contacts whose answer is known: a major donor,
a lapsed supporter, a highly engaged non-donor. Confirm the points come from the rules you expect.
This catches the case where the distribution looks healthy and the model is measuring the wrong thing.

The first pass of that analysis on one model surfaced a real defect: the top of the Fit distribution
contained organizational records that should never have been scored as individuals at all. The fix was
to exclude them, rather than to re-weight the model around their presence.

## What I would do differently

None of the three models has automated drift detection. Scores re-evaluate continuously, so the
distribution shifts as the underlying population changes, and there is no alert when the shape moves
far enough that the bands stop meaning what they meant at launch. A scheduled distribution snapshot
compared against the launch baseline would catch that, and it is the same argument as
[case study 11](11-integration-forensics.md): the model is a system in production, and systems in
production need something watching them.

Thresholds were also set loose at launch, to be tightened as real engagement accumulated. That was the
right call for the cold-start problem. It also requires somebody to actually come back and tighten
them, and that follow-up needs an owner and a date rather than an intention.

---

**Related patterns:** [Score budget discipline](../patterns/score-budget-discipline.md) ·
[Directional difference verification](../patterns/directional-difference-verification.md) ·
[Map dependencies before changing an enum](../patterns/map-dependencies-before-changing-an-enum.md)
