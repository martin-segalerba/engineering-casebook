---
title: "12 · Four production incidents, including one I caused"
parent: Case studies
nav_order: 12
---

# Four production incidents, including one I caused

**Clients:** Civic, Surgical, Signal (anonymized)
**Role:** Solutions Engineer: diagnosis, rollback, root cause, prevention
**Stack:** HubSpot, Salesforce, sandbox and production environments, private app APIs

Everything else in this casebook is work that was planned. This is the work that arrives at eleven in
the morning and decides what the rest of the day looks like.

## Why these are here together

Four incidents across three clients, over eighteen months. They have different causes and one thing in
common, which is the finding that matters more than any individual fix: **all four were detected by a
person noticing, not by a system reporting.**

## Incident 1: a sandbox task reaches production

**What happened.** Audience lists inflated past two million contacts, in a portal where the working
audiences are a fraction of that. Segmentation across the portal was wrong at once.

**Cause.** Work intended for the sandbox reached production. The environments were similar enough that
the mistake was easy to make and the result was immediately portal-wide.

**Response.** Diagnosed, rolled back, and confirmed restoration the same day. The confirmation is the
part worth naming: rollback is not the end of the incident, and "it should be fine now" is not a
finding. Verifying that list membership had actually returned to expected counts is what closed it.

**What it changed.** It fed directly into how I later handled sandbox-to-production deployment, which
became a controlled path with a pre-deploy staging step rather than an informal one. A deployment
mechanism that makes this mistake easy will have it made.

## Incident 2: my own workflow corrupts 2,900 records

**What happened.** A workflow I configured corrupted roughly 2,900 appointment records, creating
duplicates and bad associations, on the integration described in
[case study 06](06-clinical-integration-postmortem.md).

**Cause.** Mine. The workflow was misconfigured, and it ran at volume before anyone saw the output.

**Response.** Reverted the workflow, then cleaned up the duplicates it had created. Those are two jobs,
and the second is larger. Stopping the process that is producing bad records leaves you with every bad
record it already produced, and the cleanup has to distinguish what it created from what was
legitimately there.

**What I take from it.** It sits in the same portal as case study 06, which is a post-mortem of
somebody else's design producing duplicate records. I wrote that analysis, and then produced duplicate
records myself through a different mechanism. The uncomfortable and correct reading is that knowing
the failure mode is not protection against it. What protects you is testing on a bounded population
before running at volume, which is a procedure, not an insight. Nothing about my understanding of the
system prevented this, and a ten-record test would have.

## Incident 3: an import deletes 1,906 contacts

**What happened.** A data import removed 1,906 contacts that should not have been touched.

**Cause.** A missing unique identifier in the import file. Without it the import could not establish
which existing records the incoming rows corresponded to, and resolved that ambiguity destructively.

**Response.** Remediation of the affected records, with the root cause traced to the file rather than
to the importer's behavior.

**Why it belongs in this casebook.** This is the same failure as
[case study 06](06-clinical-integration-postmortem.md) and the same lesson as
[case study 02](02-crm-data-architecture.md), arriving through a third door. When identity is not
established by a reliable key, the system does something with the ambiguity, and what it does is
rarely what you would have chosen. The pattern
[idempotent upsert on external IDs](../patterns/idempotent-upsert-on-external-ids.md) exists because
this keeps happening in different costumes.

## Incident 4: a record deleted by a tool nobody suspected

**What happened.** A contact disappeared from a portal with no user action that explained it.

**Response.** Traced the deletion through API activity to a private application, identified which
installed tool was responsible, and restored the record.

**What it illustrates.** In a portal with a dozen connected applications, "who did this" is a real
question with a findable answer, and most people do not know it is findable. The API and audit
surfaces record which integration acted and when. This is the same instinct as
[provenance-first debugging](../patterns/provenance-first-debugging.md): do not reason about who
probably did it, read what actually did.

The broader finding is that every private app with write scope is a thing that can delete your data,
and portals accumulate those the way they accumulate workflows. Nobody inventories them.

## The pattern across all four

Detection was human in every case. Somebody looked at a number that seemed wrong.

Three of the four were caused by a legitimate mechanism doing exactly what it was configured to do:
a deployment moved what it was told to move, a workflow ran as configured, an import resolved
ambiguity the way importers do. There was no bug. The systems were correct and the instructions were
wrong, which is the normal shape of a production incident in a configured system and the reason
"is the software working" is the wrong first question.

The prevention that generalizes is not better configuration, since these were configured by people who
knew the systems, including me. It is:

- **Test at bounded volume before running at full volume.** Incident 2 needed ten records, not 2,900.
- **Confirm the fix rather than assuming it.** Incident 1 was only closed by verifying the counts came
  back.
- **Never let a write operation resolve identity ambiguity on its own.** Incident 3.
- **Know what is installed and what it can write.** Incident 4.

## What I would do differently

I would build detection. Every one of these was found late because nothing was watching, and in each
case the watching was cheap: an alert on list membership changing by an order of magnitude, on record
creation rates outside a normal band, on deletions above a daily threshold. None of that is difficult
and none of it existed, because monitoring is the work that feels unnecessary right up until the
morning it would have saved the day.

I made exactly that argument on a later engagement and built the monitoring there, which is
[case study 11](11-integration-forensics.md). These four incidents are why.

---

**Related patterns:** [Build the missing trigger](../patterns/build-the-missing-trigger.md) ·
[Idempotent upsert on external IDs](../patterns/idempotent-upsert-on-external-ids.md) ·
[Provenance-first debugging](../patterns/provenance-first-debugging.md)
