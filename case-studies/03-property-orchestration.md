---
title: "03 · A source-of-truth layer for 1,000+ properties from four systems"
parent: Case studies
nav_order: 3
---

# A source-of-truth layer for 1,000+ properties from four systems

**Client:** Civic (US state-level civic advocacy nonprofit)
**Role:** Solutions Engineer: problem framing, architecture options, costing, recommendation
**Deliverable:** an architecture proposal with three costed options and an implementation path

This case study is about a decision rather than a build. The value is in how the options were framed,
costed and chosen.

## Context

After the data architecture work in [case study 02](02-crm-data-architecture.md), Civic's HubSpot
portal held over 1,000 contact properties and 130 household properties, populated from four
independent sources: web forms and manual entry, a canvassing application, a voter file, and a
consumer data file.

Many of those fields overlap. The same underlying concept exists several times under different names,
on different scales, with different freshness. The previous project unified the schema and removed
duplicate contacts. It did not decide which value wins when two sources disagree.

## Problem

Stated as the question the client actually asked: "why is this project necessary if we just finished a
data architecture project?"

The honest answer is that the earlier work covered structure and deduplication, and this covers logic
and governance. Unifying the schema puts four values for someone's income range into four
well-named fields. It does not say which one to trust, who is allowed to change it, or how you would
find out later why it says what it says.

Three consequences were already visible: conflicting values with no resolution rule, no lineage when
someone asks who set a value and when, and no way to expose a clean field to a non-administrator
without exposing the four raw ones behind it.

There was also a forward-looking driver. The organization was considering a multi-brand setup on one
portal. Without an orchestration layer, each brand would edit the same fields independently, dashboards
mixing brands would contradict each other, and there would be no way to control what each brand can
see or change.

## Constraints

- **Non-administrators must see one field, not four.** The complexity is real and has to be hidden
  behind a canonical value.
- **Every decision must be auditable.** Which source won, when, and under which rule.
- **Rules change.** Source precedence is a business decision that will be revised, so it cannot be
  compiled into code that requires a developer to adjust.

## The three options

Each was specified to the same depth: technical description, considerations, honest pros and cons, and
a task-level effort breakdown by role.

### Option 1: native workflows with custom code actions

Logic runs inside HubSpot. Workflows with JavaScript actions compare source values per property group,
apply precedence, and write the canonical field.

Nothing to host and quick for a small property set, and non-developers can inspect it. Complexity
grows sharply with property count, there is no central audit log, and it stops being practical beyond
roughly 100 orchestrated properties. Against 1,000 properties it is a prototype rather than a solution.
**Roughly 47 hours.**

### Option 2: external middleware, the property sync orchestrator

A small external service reads from HubSpot, evaluates precedence and confidence, writes the canonical
fields back, and logs every decision. Rules, property equivalences, normalization mappings and
confidence weights live in editable configuration rather than in code. Runs on a schedule with webhooks
for real-time sources.

Centralized logic, full auditability, configurable without a deployment, reusable across clients, and
unconstrained by platform limits. It needs hosting and it raises the technical bar slightly.
**Roughly 86 hours.**

### Option 3: data lake orchestrator

All raw source data lands in a warehouse. Transformations apply normalization, precedence and
confidence, and only final values are pushed to HubSpot, which becomes a display layer.

Best scalability and transparency, enables analytics and version control, and is the right foundation
for a multi-brand platform. It carries the highest setup cost and infrastructure overhead and needs a
skill set the team does not have. **Roughly 124 hours.**

| Option | Scalability | Auditability | Infrastructure | Hours | Best fit |
|---|---|---|---|---|---|
| Native workflows | Low | Low | None | ~47 | Prototype |
| Middleware | High | High | Minimal | ~86 | Production, multi-source |
| Data lake | Very high | Very high | Yes | ~124 | Multi-brand platform stage |

## Recommendation

Start with the middleware. It balances control, scalability and maintainability while staying light on
infrastructure, and it produces the clean canonical layer in HubSpot that the client actually asked
for. The data lake gets layered on later, once the system stabilizes and analytics or brand isolation
justify it.

The part I would defend is naming the upgrade trigger in advance rather than presenting the cheapest
adequate option as the end state. The proposal says explicitly what would move the project to option 3:
brand count, source count, data volume, or compliance requirements. That converts "we will revisit
later" into a decision with a condition attached.

## The design the middleware implements

- **Per-source fields, per-property canonical field.** Each source keeps its own field. A canonical
  field holds the resolved value and is the only one non-administrators see.
- **Precedence per field, not per source.** The voter file may be authoritative for registration status
  while a canvasser is authoritative for phone number. A global source ranking cannot express that.
- **Confidence scoring with the reasoning recorded.** Each resolved value carries a score plus the
  sources, timestamps and rule that produced it, so a value can be explained and reverted.
- **Normalization before comparison.** Two sources agreeing on a concept but disagreeing on scale or
  spelling are not a conflict, and treating them as one is how precedence rules get written to solve
  problems that were really formatting.

## What I left open, deliberately

The proposal closes with eleven unresolved questions rather than a plan pretending to be complete:
property scope for phase one, property lifecycle ownership, onboarding a new source, static versus
adaptive confidence scoring, retention and revert, run frequency and acceptable latency, per-brand
visibility and edit rights, what happens when no source has a valid value, credential storage and
admin access, monitoring KPIs and failure alerting, and the migration trigger toward option 3.

Most of those are business decisions wearing technical clothes. Answering them myself would have been
faster and would have produced a system encoding my assumptions about the client's governance. Listing
them made the client's ownership of those decisions explicit before any of them got made by default.

## Outcome

The proposal reframed the request from "clean up our data again" into an architecture decision with
costed options, a recommendation with reasoning, and a named condition for revisiting it. The
justification section addressed the client's real question directly, which is what made the rest of the
document readable to a non-technical stakeholder.

## What I would do differently

The three options are costed but not risk-adjusted, and the 86-hour middleware estimate assumes rules
that are already known. In practice, defining precedence across 1,000 properties is discovery work, and
that discovery is the part most likely to overrun. I would present the mapping phase as a separate,
smaller, time-boxed engagement whose output is the input to a firmer estimate for the build, rather than
folding an unknown into a single number.

---

**Related patterns:** [Provenance-first debugging](../patterns/provenance-first-debugging.md)
