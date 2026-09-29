---
title: "04 · Replacing a vendor integration with a webhook you control"
parent: Case studies
nav_order: 4
---

# Replacing a vendor integration with a webhook you control

**Client:** Civic (US state-level civic advocacy nonprofit)
**Role:** Solutions Engineer: integration design, rebuild, team documentation
**Stack:** HubSpot forms, workflows and webhooks, a third-party advocacy action platform

Short-form case study. The build is small. The reason it is here is what happened to it five months
after it shipped.

## Context

Civic runs advocacy campaigns through a third-party action platform where supporters contact their
legislators. Every action taken on that platform has to arrive in HubSpot attributed to the campaign
that produced it, because campaign performance is how the organization decides where to spend.

## Problem

The two systems had no shared campaign identifier. Submissions arrived, but nothing on the record said
which campaign they came from, so attribution was reconstructed afterwards from timestamps and page
URLs. That reconstruction is guesswork, and it gets worse as more campaigns run at once.

## What I designed

**A hidden campaign identifier carried on the form.** Each HubSpot form carries a hidden field whose
default value is the action platform's own campaign ID. The submission therefore arrives already
holding the join key. Attribution stops being an inference and becomes a property on the record, set
at the moment of submission.

The operational consequence is what made it worth doing: launching a campaign became cloning a form
and pasting one ID into one field. That is something the marketing team does without an engineer.

**A webhook workflow instead of the vendor's own integration.** Rather than rely on the platform's
packaged integration, I found its campaign submit API and drove it from a HubSpot workflow with a
webhook action. That put the contract between the two systems in a place I could read, version and
change, instead of inside a connector whose behavior I could only observe.

**A documented procedure rather than tribal knowledge.** The whole thing ships as an eight-step SOP
with a test submission and a rollback, because the people who run it are campaigners.

## The part worth writing down

In February 2026, five months after go-live, the platform changed its API without notice. No
deprecation warning, no changelog entry, no email. Submissions simply stopped arriving.

Because the integration was a webhook call I owned rather than a connector I rented, the fix was
mine to make. I rebuilt the call as a custom code action and had it running again the next day.

The generalizable point is not "build it yourself." It is that when you depend on an external
system, you should know which failures you are able to fix and which ones leave you filing a support
ticket and waiting. A packaged connector would have failed exactly as fast, and the recovery would
have been someone else's queue. Choosing the more manual integration bought a recovery time I
controlled, and that only paid off once, which is the usual way this kind of decision pays off.

## Trade-offs

**A hidden field on a cloneable form, rather than a deeper API integration.** The field can be broken
by anyone editing the form, and a deeper integration would be sturdier. It would also put every new
campaign back in an engineering queue. For a team launching campaigns weekly, the cloneable form won,
and the SOP's verification step exists to catch the breakage that choice allows.

**Owning the API call rather than using the vendor's integration.** More to build and more to
maintain, and I carry the burden when the vendor changes something. That burden turned out to be the
asset.

## Outcome

Every advocacy submission arrives attributed to a campaign, with the join key present on the record at
submission time rather than reconstructed later. Campaign launches stopped requiring engineering. When
the vendor broke the contract without telling anyone, the integration was back the next day.

## What I would do differently

Nothing detected the outage. It was noticed because someone looked at submission counts and found them
at zero. An integration that depends on an external API should ship with a check that alerts when
expected traffic stops arriving, which is a small amount of work next to the build itself. I made that
argument properly on a later engagement and built the monitoring there, which is
[case study 11](11-integration-forensics.md).

---

**Related patterns:** [Build the missing trigger](../patterns/build-the-missing-trigger.md) ·
[Provenance-first debugging](../patterns/provenance-first-debugging.md)
