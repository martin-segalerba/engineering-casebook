---
title: Case studies
nav_order: 2
has_children: true
---

# Case studies

Thirteen pieces of production work, written to a common shape: context, problem, constraints, what I
designed, trade-offs, outcome, how it was verified, and what I would do differently.

All clients are anonymized. See [the anonymization policy](../ANONYMIZATION.md) for exactly what was
removed and what was kept.

| # | Case study | Client | Headline |
|---|---|---|---|
| 01 | [Identity disaggregation at two-million-contact scale](01-identity-disaggregation.md) | Civic | 191 merged super-contacts reconstructed into 9,916 people, disaggregation errors audited to ~0 |
| 02 | [Unifying two large datasets into one CRM schema](02-crm-data-architecture.md) | Civic | 2.09M contacts joined to a 3.05M-row dataset with no shared key |
| 03 | [A source-of-truth layer for 1,000+ properties](03-property-orchestration.md) | Civic | Three costed architectures, a recommendation, and a named upgrade trigger |
| 04 | [Replacing a vendor integration with a webhook you control](04-advocacy-attribution.md) | Civic | The vendor changed its API without notice; the integration was back the next day |
| 05 | [Rebuilding a 200-workflow portal](05-portal-rebuild.md) | Relief | Suppression rebuilt as expiring tasks; one list definition changed in place instead of across 182 references |
| 06 | [Post-mortem of a clinical to CRM integration](06-clinical-integration-postmortem.md) | Surgical | Why it produced duplicate records in production, and the specification that replaced it |
| 07 | [Bidirectional CRM sync across three systems](07-bidirectional-crm-sync.md) | Signal | Conflict resolution declared per field rather than per system |
| 08 | [Collapsing 203 copies of a workflow into one](08-workflow-consolidation.md) | Relief | 753 property writes mapped across 203 copies, replaced by one workflow; 212 legacy workflows retired |
| 09 | [A static analyzer for a CRM automation graph](09-automation-static-analysis.md) | Own tool | 25 rules over a def-use graph, three analysis passes, installed on client portals by permission |
| 10 | [Executing a migration of millions of records](10-migration-at-scale.md) | Civic | 56,830 merges under API throttling, gigabyte files chunked, and the bug that ate leading zeros |
| 11 | [Diagnosing silent integration failures, and building the alarm](11-integration-forensics.md) | Counsel | Three root causes separated, and monitoring built because the platform has no trigger for silence |
| 12 | [Four production incidents, including one I caused](12-incident-response.md) | Multiple | All four found by a person noticing, which is the actual finding |
| 13 | [Three scoring models in production](13-scoring-models.md) | Multiple | Two axes rather than one, and a budget that has to sum to the ceiling |

## Where to start

**If you want the hardest technical problem**, read [01](01-identity-disaggregation.md).
**If you want to see how I build software rather than specify it**, read
[09](09-automation-static-analysis.md).
**If you want to know how I behave when things go wrong**, read
[06](06-clinical-integration-postmortem.md) and [12](12-incident-response.md), which are a post-mortem
of someone else's failure and an account of four of my own incidents.
