---
title: "04 · Advocacy attribution across an external action platform"
parent: Case studies
nav_order: 4
---

# Advocacy attribution across an external action platform

**Client:** Civic (US state-level civic advocacy nonprofit)
**Role:** Solutions Engineer: integration design, workflow consolidation, team documentation
**Stack:** HubSpot forms and workflows, New/Mode advocacy platform, paid acquisition vendors

Short-form case study. The build is small and the interesting part is the consolidation it enabled and
the failure mode that nearly shipped with it.

## Context

Civic runs advocacy campaigns through an external action platform where supporters contact their
legislators. Supporters arrive from organic traffic and from paid list-buy vendors, and every action
taken on the platform needs to land in HubSpot attributed to the right campaign, the right advocacy
topic and the right acquisition source.

## Problem

Two things were coupled that should not have been.

The action platform and HubSpot had no shared campaign identifier, so submissions arrived without a
reliable way to say which campaign produced them.

Separately, the automation pattern had one workflow per form. Launching a new paid vendor meant
building new automation, which put a marketing task behind an engineering queue and grew the workflow
count with every campaign. That is the same accumulation pattern documented at a much later stage in
[case study 05](05-portal-rebuild.md).

## What I designed

**A hidden campaign identifier on the form.** Each HubSpot form carries a hidden field whose default
value is the action platform's unique campaign ID. The submission arrives already carrying the join
key, so attribution is a property on the record rather than something reconstructed later from
timestamps and page URLs.

**One routing workflow replacing one workflow per form.** A single lead-generation routing and source
attribution workflow reads the transit fields from the submission and branches on vendor. Adding a paid
vendor became: add the value to the vendor property, add the same value to the source properties, add
it to the paid branch condition. Launching a vendor went from building automation to adding a dropdown
value, which is the entire reason the consolidation was worth doing.

**Fiscal-year fields isolated by design.** Paid-lead tracking flags carry a fiscal year. Those pieces
were kept separate from the routing logic so the annual rollover is a checklist that adds new
properties and repoints a handful of actions, rather than a rebuild. Prior-year properties are left
untouched as the historical reporting record.

## The failure mode worth writing down

The platform copies values between properties by **internal name**, not by label. If the same option is
`VendorName` in one property and `Vendorname` in another, the copy silently produces nothing. It does
not error, does not warn, and does not appear in any log.

This is what held up the original go-live. The visible symptom is that some fields on a submission are
populated and others are empty, which looks like a broken integration and is actually a one-character
casing difference in an internal value that nobody looks at because the label reads correctly.

The procedure now says: when adding a vendor, set the label and the internal value deliberately, write
the internal value down, and use the identical string across all three properties. The verification
step is to submit a test and read the contact's property history, checking that all three transit
fields were written within a minute of each other. If one gained a value and another did not, the
internal names do not match.

## Trade-offs

**Hidden field on a cloned form, rather than an API integration.** Cloning a form and setting one
hidden default is something the marketing team does without an engineer. An API integration would be
more robust against someone editing the form, and it would put every new campaign back in the
engineering queue. For a team launching campaigns weekly, the cloneable form won.

**One branching workflow rather than one per vendor.** The single workflow is larger and its branch
tree needs reading. It is also the only place in the portal that defines what "paid" means, which is
worth more than the readability of any individual branch.

## Outcome

Every advocacy submission arrives attributed to a campaign, a topic and an acquisition source, with the
join key present on the record at submission time. Vendor launches stopped requiring new automation.
The whole thing is documented as an eight-step procedure with a test and a rollback, because the people
running it are marketers rather than engineers.

The rollback is worth noting: to remove a vendor, take the value out of the paid branch condition
first, so its submissions route as non-paid, which is the safe default, and only then remove the
property options. Never delete an option that live contacts still hold, because the stored value stays
on the record while filters stop matching it.

## What I would do differently

The internal-name matching requirement is enforced by a written procedure and a human remembering to
follow it, which is a control that works until someone is in a hurry. A scheduled check comparing the
option sets of the three properties and alerting on divergence would catch it without depending on
anyone's care, and it would take an afternoon to build.

---

**Related patterns:** [Provenance-first debugging](../patterns/provenance-first-debugging.md)
