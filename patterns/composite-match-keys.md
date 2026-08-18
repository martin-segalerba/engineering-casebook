---
title: Composite match keys
parent: Patterns
---

# Composite match keys

**Rule.** When two systems hold the same people and share no identifier, construct one: concatenate a
fixed set of identifying attributes into a single string, compute it identically on both sides, and
join on exact equality. Store it as a visible property rather than computing it inside a matching
script.

**Why a stored property.** The key becomes inspectable. A non-technical operator can open a record,
read the key, and understand why it matched or did not. Matching logic that only exists inside a script
is a black box to the people who have to trust its output.

**Precedence matters more than the key itself.** A real identifier always beats a constructed one. The
order is: hard identifier where present, composite key as fallback, flagged orphan where neither
works. Force-matching orphans on weaker criteria raises the coverage number and lowers the truth.

**The failure this prevents:** fuzzy matching. Fuzzy name-and-address matching is the mechanism that
produced the damage in [case study 01](../case-studies/01-identity-disaggregation.md), where distinct
people were merged into single records because they looked similar enough. An exact join on an explicit
key is auditable and its failures are visible; a similarity threshold's failures are not.

**The failure it introduces:** nulls. A key built from five components where one is missing produces a
string that matches nothing, silently. Handle missing components explicitly, and report key
completeness as a distribution before you import, so the population that structurally cannot match is
known up front rather than discovered as an unexplained orphan count.

**Evidence:** [Case study 02](../case-studies/02-crm-data-architecture.md), joining a 2.09M-contact CRM
against a 3.05M-row dataset with no shared primary key.
