---
title: Home
nav_order: 1
---

# Systems Casebook

Case studies in CRM data architecture, identity resolution and system integration, from production
work as a Solutions Engineer between 2025 and 2026.

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
| System integration and sync design | [06](case-studies/06-clinical-integration-postmortem.md), [07](case-studies/07-bidirectional-crm-sync.md), [04](case-studies/04-advocacy-attribution.md) |
| Rebuilding accumulated systems in place | [05](case-studies/05-portal-rebuild.md) |
| Architecture selection and costing | [03](case-studies/03-property-orchestration.md) |
| Post-mortem and root cause analysis | [06](case-studies/06-clinical-integration-postmortem.md) |

## The case studies

| # | Case study | Headline |
|---|---|---|
| 01 | [Identity disaggregation at two-million-contact scale](case-studies/01-identity-disaggregation.md) | An automated dedup had fused hundreds of distinct people into single records. Built an engine that reconstructs them from merge history: 191 super-contacts into 9,916 people, with over-split, misattribution and import collisions all audited to ~0 |
| 02 | [Unifying two large datasets into one CRM schema](case-studies/02-crm-data-architecture.md) | Joining 2.09M CRM contacts to a 3.05M-row dataset with no shared primary key, and sequencing the import so a failure partway through is recoverable |
| 03 | [A source-of-truth layer for 1,000+ properties](case-studies/03-property-orchestration.md) | Three architectures costed at 47, 86 and 124 hours, with a recommendation and a stated condition for moving to the next one |
| 04 | [Advocacy attribution across an external platform](case-studies/04-advocacy-attribution.md) | One routing workflow replacing one workflow per form, and the silent internal-name mismatch that nearly shipped with it |
| 05 | [Rebuilding a 200-workflow portal](case-studies/05-portal-rebuild.md) | Suppression rebuilt as expiring tasks so holds release themselves, and a list definition referenced in 182 places changed with one edit |
| 06 | [Post-mortem of a clinical to CRM integration](case-studies/06-clinical-integration-postmortem.md) | Work that failed in production. Why a flattened model with editable identifiers generated duplicate records, and the specification that replaced it |
| 07 | [Bidirectional CRM sync across three systems](case-studies/07-bidirectional-crm-sync.md) | Conflict resolution declared per field rather than per system, across several hundred fields |

## Patterns

The [patterns library](patterns/index.md) is the reusable half: nine rules extracted from the work above, each
with the evidence behind it and the failure it prevents. Start there if you want the engineering
without the client context.

## A note on what is here

Two of these case studies describe work that went wrong. [06](case-studies/06-clinical-integration-postmortem.md)
is a post-mortem of an integration that shipped and produced bad data.
[01](case-studies/01-identity-disaggregation.md) exists because an automated process was allowed to
merge records irreversibly on soft matching. Both are included on purpose. The verification sections
and the "what I would do differently" sections are the ones I would read first.

---

*Martín Segalerba*
