---
title: Case studies
nav_order: 2
has_children: true
---

# Case studies

Seven pieces of production work, written to a common shape: context, problem, constraints, what I
designed, trade-offs, outcome, how it was verified, and what I would do differently.

All clients are anonymized. See [the anonymization policy](../ANONYMIZATION.md) for exactly what was
removed and what was kept.

| # | Case study | Client | Headline |
|---|---|---|---|
| 01 | [Identity disaggregation at two-million-contact scale](01-identity-disaggregation.md) | Civic | 191 merged super-contacts reconstructed into 9,916 people, disaggregation errors audited to ~0 |
| 02 | [Unifying two large datasets into one CRM schema](02-crm-data-architecture.md) | Civic | 2.09M contacts joined to a 3.05M-row dataset with no shared key |
| 03 | [A source-of-truth layer for 1,000+ properties](03-property-orchestration.md) | Civic | Three costed architectures, a recommendation, and a named upgrade trigger |
| 04 | [Advocacy attribution across an external platform](04-advocacy-attribution.md) | Civic | Vendor launch reduced from building automation to adding a dropdown value |
| 05 | [Rebuilding a 200-workflow portal](05-portal-rebuild.md) | Relief | Suppression rebuilt as expiring tasks; one list definition changed in place instead of across 182 references |
| 06 | [Post-mortem of a clinical to CRM integration](06-clinical-integration-postmortem.md) | Surgical | Why it produced duplicate records in production, and the specification that replaced it |
| 07 | [Bidirectional CRM sync across three systems](07-bidirectional-crm-sync.md) | Signal | Conflict resolution declared per field rather than per system |
