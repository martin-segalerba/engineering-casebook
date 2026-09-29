---
title: Home
nav_order: 1
---

# Systems Casebook

Thirteen case studies in CRM data architecture, identity resolution and system integration, from
production work as a Solutions Engineer between 2025 and 2026.

Every client is anonymized. Scale figures, error rates and technical decisions are unchanged, because
they are the part worth reading. The [anonymization policy](ANONYMIZATION.md) states exactly what was
removed and what was kept.

## What this work looks like

Most of it starts the same way: a CRM holding millions of records, fed by several systems that
disagree with each other, where nobody can say which value is true or why a record is in the state it
is in. The engineering is identity, precedence and verification, and the constraint is that these are
live systems that cannot be taken down while being repaired.

| Area | Where to look |
|---|---|
| Identity resolution and record reconstruction | [01](case-studies/01-identity-disaggregation.md), [02](case-studies/02-crm-data-architecture.md) |
| Data architecture and source-of-truth design | [02](case-studies/02-crm-data-architecture.md), [03](case-studies/03-property-orchestration.md) |
| Building tools | [09](case-studies/09-automation-static-analysis.md), [11](case-studies/11-integration-forensics.md) |
| Operating at scale | [10](case-studies/10-migration-at-scale.md) |
| System integration and sync design | [06](case-studies/06-clinical-integration-postmortem.md), [07](case-studies/07-bidirectional-crm-sync.md), [04](case-studies/04-advocacy-attribution.md) |
| Rebuilding accumulated systems in place | [05](case-studies/05-portal-rebuild.md), [08](case-studies/08-workflow-consolidation.md) |
| Debugging and incident response | [11](case-studies/11-integration-forensics.md), [12](case-studies/12-incident-response.md), [06](case-studies/06-clinical-integration-postmortem.md) |
| Modeling and scoring | [13](case-studies/13-scoring-models.md) |
| Architecture selection and costing | [03](case-studies/03-property-orchestration.md) |

## The case studies

| # | Case study | Headline |
|---|---|---|
| 01 | [Identity disaggregation at two-million-contact scale](case-studies/01-identity-disaggregation.md) | An automated dedup had fused hundreds of distinct people into single records. Built an engine that reconstructs them from merge history: 191 super-contacts into 9,916 people, with over-split, misattribution and import collisions all audited to ~0 |
| 02 | [Unifying two large datasets into one CRM schema](case-studies/02-crm-data-architecture.md) | Joining 2.09M CRM contacts to a 3.05M-row dataset with no shared primary key, and sequencing the import so a failure partway through is recoverable |
| 03 | [A source-of-truth layer for 1,000+ properties](case-studies/03-property-orchestration.md) | Three architectures costed at 47, 86 and 124 hours, with a recommendation and a stated condition for moving to the next one |
| 04 | [Replacing a vendor integration with a webhook you control](case-studies/04-advocacy-attribution.md) | Five months after go-live the vendor changed its API with no notice and submissions stopped. Because the call was mine rather than a rented connector, it was running again the next day |
| 05 | [Rebuilding a 200-workflow portal](case-studies/05-portal-rebuild.md) | Suppression rebuilt as expiring tasks so holds release themselves, and a list definition referenced in 182 places changed with one edit |
| 06 | [Post-mortem of a clinical to CRM integration](case-studies/06-clinical-integration-postmortem.md) | Work that failed in production. Why a flattened model with editable identifiers generated duplicate records, and the specification that replaced it |
| 07 | [Bidirectional CRM sync across three systems](case-studies/07-bidirectional-crm-sync.md) | Conflict resolution declared per field rather than per system, across several hundred fields |
| 08 | [Collapsing 203 copies of a workflow into one](case-studies/08-workflow-consolidation.md) | 203 copies performing 753 property writes onto 14 properties are not 203 problems. They are one missing layer of indirection. 212 legacy workflows retired the day after cutover |
| 09 | [A static analyzer for a CRM automation graph](case-studies/09-automation-static-analysis.md) | A CRM automation portal is a graph, so cycle detection, dead code and fan-out analysis all apply. Built the analyzer: 25 rules over two inverted indexes, three passes, and clients authorized installing it on production portals |
| 10 | [Executing a migration of millions of records](case-studies/10-migration-at-scale.md) | The operational half of case 02. 56,830 merges under API throttling, gigabyte files chunked correctly, and a spreadsheet quietly eating the leading zeros off identifiers |
| 11 | [Diagnosing silent integration failures, and building the alarm](case-studies/11-integration-forensics.md) | Three separate root causes behind one "flaky integration". The platform has no trigger for the absence of an event, so the monitoring had to be built outside it |
| 12 | [Four production incidents, including one I caused](case-studies/12-incident-response.md) | A sandbox task reaching production, my own workflow corrupting 2,900 records, an import deleting 1,906 contacts, and a record deleted by a tool nobody suspected. All four found by a person noticing |
| 13 | [Three scoring models in production](case-studies/13-scoring-models.md) | Why every model here has two axes instead of one, why group limits must sum exactly to the ceiling, and how to run the empirical work under a client AI policy rather than around it |

## Patterns

The [patterns library](patterns/index.md) is the reusable half: thirteen rules extracted from the work
above, each with the evidence behind it and the failure it prevents. Start there if you want the
engineering without the client context.

## Technical leadership

[A separate page](leadership.md) covers reviewing other people's delivery work, the tooling built for
the team, and writing the framework for how a services organization adopts AI agents.

## A note on what is here

Several of these describe work that went wrong.
[06](case-studies/06-clinical-integration-postmortem.md) is a post-mortem of an integration that
shipped and produced bad data. [12](case-studies/12-incident-response.md) is four production incidents,
one of them mine. [01](case-studies/01-identity-disaggregation.md) exists because an automated process
was allowed to merge records irreversibly on soft matching.

They are included on purpose. The verification sections and the "what I would do differently" sections
are the ones I would read first.

---

*Martín Segalerba*
