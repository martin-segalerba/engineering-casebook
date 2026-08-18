---
title: "07 · Bidirectional CRM sync across three systems"
parent: Case studies
nav_order: 7
---

# Bidirectional CRM sync across three systems

**Client:** Signal (B2B identity-resolution SaaS)
**Role:** Solutions Engineer: integration architecture and field-level mapping
**Stack:** Salesforce, HubSpot, a third-party intent and propensity data platform, native and custom sync

## Context

Signal ran sales in Salesforce and marketing in HubSpot, with a third platform supplying intent and
propensity signals on accounts and contacts. Three systems, each of which believed it owned the
customer record.

The engagement was to define how those systems share data: which records cross the boundary, in which
direction, and what happens when both sides have a value and they differ.

## Problem

Bidirectional sync is usually configured as a system-level setting, some variant of "keep these
objects in sync". That framing fails as soon as the two systems have genuinely different authority
over different attributes of the same record.

Sales owns opportunity data and account ownership in Salesforce. Marketing owns engagement, email
consent and campaign attribution in HubSpot. The intent platform owns propensity scores and website
intent signals, and neither CRM should ever write those back. A single sync direction for the contact
object cannot express any of that, and choosing one direction means silently overwriting whichever
side was not chosen.

Second problem: without scoping, everything syncs. Pushing every marketing contact into Salesforce
consumes licensed storage and fills the sales system with records nobody will ever work.

## Constraints

- **Salesforce is the system of record for the sales objects.** Accounts, opportunities and their
  ownership are decided there.
- **The intent platform is one-directional by nature.** It emits signals and consumes nothing.
- **Contact volumes are asymmetric.** Marketing holds far more contacts than sales should ever see.

## What I designed

### Sync direction declared per object and per trigger

Rather than a global toggle, each object gets an explicit matrix of trigger, action, target and
direction. Contact creation, contact update, lead creation, lead conversion, opportunity creation and
opportunity update are each a separate row with its own direction, and each one is either active,
planned or conditional. Deletion is handled deliberately, and mostly by not propagating: a delete in
one system does not delete in the other unless the row says so.

Writing it as a matrix rather than prose is what surfaced the conflicts. Two rows wanting to write the
same field from opposite directions is visible in a table and invisible in a paragraph.

### Conflict resolution declared per field

The core of the deliverable is a field-level mapping across contact, company and deal objects, several
hundred fields in total, each carrying one of four explicit rules:

| Rule | Meaning | Typical use |
|---|---|---|
| Two way | Either side may update; latest write wins | Shared descriptive fields such as phone or title |
| Prefer one side unless blank | The authoritative system wins, but does not overwrite with nothing | Firmographics owned by one system but sometimes only present in the other |
| Always to one side | One-directional, overwrites unconditionally | Intent scores, propensity data, lifecycle owned by one system |
| Do not sync | Field stays local | Internal scoring, system fields, anything meaningless across the boundary |

"Prefer unless blank" is the rule that does the most work in practice. Straight one-directional sync
from the authoritative system erases good data in the other system whenever the authority happens to
have nothing, and that erasure looks like a sync bug rather than a policy decision.

### Inclusion scoping at the boundary

Which records cross is defined by an explicit inclusion list rather than by syncing everything and
filtering later. That keeps the sales system populated only with records that meet the qualification
criteria, and it makes the boundary itself reviewable: the question "why is this contact in
Salesforce" has an answer that is a rule rather than an accident.

### Intent data lands as its own field namespace

Propensity and intent properties come in under a consistent prefix, are always one-directional, and
associate to the record types the vendor actually resolves. Keeping them namespaced means they are
never confused with first-party data, and the "always to one side, never write back" rule can be
applied to the whole namespace instead of field by field.

## Trade-offs

**Per-field rules instead of per-object rules.** This is more work up front, several hundred decisions
rather than three, and it is a document that has to be maintained. The alternative is discovering the
conflicts in production as unexplained data loss, which costs more and costs it later.

**Explicit inclusion scoping instead of syncing everything.** Sync-everything is simpler to configure
and defers the decision. It also fills a licensed system with records that will never be worked, and
un-syncing later is harder than not syncing now.

**Lead conversion handled as its own case.** Salesforce lead conversion is not an update, it is a
record transformation, and treating it as an update produces orphans on the HubSpot side. It gets its
own row in the matrix rather than being folded into contact update.

## Outcome

The deliverable is an architecture diagram and a field-level mapping that together answer, for any
field on any of the three core objects, which system decides its value and what happens when they
disagree. That document is the reference the build works from, and it is also what makes the
integration auditable afterwards, since every observed sync behavior traces to a row someone approved.

## What I would do differently

The mapping is a static document, and a static document describing several hundred fields drifts from
the running configuration the moment anyone edits the sync in either platform. I would keep the rules
in a machine-readable file that the integration reads directly, so the specification and the behavior
cannot diverge, which is the same argument that led to configuration-driven rules in
[case study 03](03-property-orchestration.md).

---

**Related patterns:** [Composite match keys](../patterns/composite-match-keys.md) ·
[Idempotent upsert on external IDs](../patterns/idempotent-upsert-on-external-ids.md)
