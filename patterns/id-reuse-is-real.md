---
title: ID reuse is real
parent: Patterns
---

# ID reuse is real

**Rule.** Before trusting a crosswalk between two datasets, test it against an attribute the join does
not use. A structurally perfect join can still be semantically wrong.

**The case.** A crosswalk linking a voter file to a consumer file on state voter ID was a perfect
one-to-one bijection across 2,401,437 rows. No duplicates, no orphans, nothing a schema check would
flag. It was used to propose merging roughly 698 pairs of records as the same person.

Testing the pairs against name and address before merging them: 64% agreed, 30% had different
addresses, and 5% were entirely different people. Roughly 3.6% of state voter IDs had been reassigned
to a different person over time.

The crosswalk table was not broken. It correctly recorded which IDs corresponded to which. The
assumption that "same ID means same person" was what failed, and that assumption is invisible inside
the join.

**The fix.** Two parts.

Confirm every bridge against an independent attribute. Here that was birth year, falling back to last
name where birth year was missing. Roughly 93% of rows had a populated birth year, and it separates
same-name people cleanly: 87 records sharing one common name resolved to 60 distinct birth years. A
mismatch rejects the bridge, which removes the wrong links while keeping the real ones.

Anchor on an identifier that is provably never reused wherever one exists. In this dataset one
identifier was unique across all 3,048,448 rows with zero reuse, and it was the correct anchor whenever
present.

**Generalize it.** Identifiers issued by an authority get recycled, reassigned and re-purposed:
government IDs, employee numbers, phone numbers, email addresses, seat numbers. Uniqueness at a point
in time is not uniqueness over time. Ask whether a key is unique across the whole history of the
dataset, and prove it rather than assuming it.

**Evidence:** [Case study 01](../case-studies/01-identity-disaggregation.md) and
[case study 02](../case-studies/02-crm-data-architecture.md).
