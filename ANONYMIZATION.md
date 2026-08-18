---
title: Anonymization policy
nav_order: 90
---

# Anonymization policy

Every case study in this repository describes real production work done under a services agreement.
None of it names a client. This page states exactly what was removed and what was kept, so a reader
can calibrate how much to trust the parts that remain.

## What is removed

- **Client names, brands and logos.** Clients are referred to by a codename and a one-line sector
  descriptor.
- **Account and portal identifiers**, and every URL pointing at a client asset.
- **Asset names as written** (workflows, lists, forms, campaigns, pipelines, journeys. Where a name
  carries the engineering point, it is paraphrased.
- **Client-branded property internal names.** A property that embedded a client's name in its slug is
  renamed to its generic equivalent, consistently, across every page.
- **Colleague and stakeholder names.**
- **Every real record used as a debugging example.** The source documentation for the identity work
  named actual individuals and an actual email address, because that is what debugging looks like.
  Those are living people's personal data. They appear here as `Contact A` and `person@example.com`.
- **Screenshots.** The original walkthrough documentation contains screen captures showing client
  names and campaign copy. None are reused. Where an image was load-bearing, it was redrawn as a
  diagram.

## What is kept

- **Vendor and platform names.** HubSpot, Salesforce, Open Dental, L2, Aristotle, New/Mode. These
  describe the stack, not the client. Each has a large customer base, and naming them is what makes
  the technical decisions legible to anyone who has worked with them.
- **Generic API and schema identifiers** that carry the engineering point: `hs_merged_object_ids`,
  `sourceType`, `TreatPlanNum`, `PatNum`. These are vendor surface area, documented publicly by the
  vendors themselves.
- **Aggregate scale figures.** Record counts, match rates, error rates, reference counts. These are
  the evidence. An aggregate over millions of records identifies no organization.

## What is not in this repository at all

Client source code. The integration described in case study 06 exists as a deployed application whose
archive contains live credentials and production logs. It was never a candidate for publication, no
file from it is quoted, and nothing here was derived from reading it. That case study is written from
specification and post-mortem documents authored by me.

Anything under `.private/`: the codename mapping, the original source documents, the project
timeline and logged hours. That directory is listed in `.gitignore` and exists only on my machine. It
is mentioned here rather than hidden, because a reader should know the redaction was a deliberate
policy and not an accident of what happened to be at hand.

## Codenames

| Codename | Sector |
|---|---|
| **Civic** | US state-level civic advocacy nonprofit: voter mobilization and economic-justice campaigning |
| **Relief** | US affiliate of an international humanitarian and anti-poverty nonprofit |
| **Surgical** | Multi-location oral and maxillofacial surgery group |
| **Signal** | B2B identity-resolution SaaS company |

Three case studies share the Civic engagement. They are written to be read independently, but the
codename lets them cross-reference where the work genuinely built on itself.
