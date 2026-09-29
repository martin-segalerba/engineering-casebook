---
title: Technical leadership
nav_order: 4
---

# Technical leadership

The [case studies](case-studies/) are systems I designed and built. This page covers the other half of
the job: reviewing other people's work, building tools the team used, and deciding how a delivery
organization should adopt a new class of technology.

It is not written to the case-study template, because none of it is a project with a cutover date.

## Reviewing and supervising delivery work

I designed solutions, built the technical and custom-coded pieces, and delegated the rest to
implementation specialists with written briefs, then signed off on their work.

Over eighteen months that came to **484 comments across 267 tasks, 186 of which were assigned to other
people**. The volume is only interesting because of what it forced: at that rate, review stops being
something you do when you have time and becomes a process that either has a shape or produces
inconsistent results.

What I did about it:

- **Briefs before work, not questions after.** A task assignment with the context, the constraints and
  the acceptance criteria written down is the difference between reviewing an implementation and
  discovering a misunderstanding.
- **A structured log of QA incidents rather than one-off corrections.** Each defect recorded, then
  analyzed for patterns. Correcting the same class of mistake in three people separately is a
  training gap being handled as three individual errors.
- **Feedback delivered from that analysis**, so it addressed recurring causes rather than individual
  instances.

The reason this matters to an engineering reader: the specialists were building the things my designs
depended on. A design that only works when its author implements it is not a design.

## Tooling for the team

Beyond [the audit tool](case-studies/09-automation-static-analysis.md), most of what I built for
internal use existed to make an expert judgment repeatable by someone who was not the expert.

- **Project scoping from the standard rate card**, so estimates came out consistent rather than
  reflecting who wrote them.
- **Task briefs to a standard structure**, which is the delegation discipline above, encoded.
- **Automation diagramming**, turning a workflow into a readable architecture diagram, because the
  archaeology described in case study 09 is not a good use of anyone's morning.
- **Naming conventions**, with the convention enforced by tooling rather than by remembering it.

The through-line is that a convention nobody can follow without effort is a convention that decays. If
a standard matters, it should be cheaper to follow than to ignore.

## Deciding how a delivery organization adopts AI agents

The substantive piece of governance work I did was a formal proposal for the approved use of AI agents
in delivery work, which became an approved internal toolkit and one of the organization's quarterly
objectives.

The problem it addressed is specific. Agent tooling arrives in a services organization from the bottom
up: individuals adopt it because it works, without a shared position on client data, on what may be
delegated, or on what must still be reviewed by a person. That produces two bad outcomes at once.
Practice diverges, so quality depends on who picked up which tool. And the organization has no answer
when a client asks what touched their data, which is a question clients had already started asking.

The proposal took a position rather than describing options: which categories of work agents are
appropriate for, what review a human owes before anything reaches a client, and how client data
constraints are handled, including the case where a client's own policy is stricter than the
organization's. [Case study 13](case-studies/13-scoring-models.md) has a worked example of navigating
exactly that.

The part I would argue for again is that it was written as an **approval framework rather than a
restriction list**. A policy that only says no gets routed around by people trying to do their jobs. A
policy that says what is approved, and how to get something approved that is not, gets followed.

I also owned a second objective in the same cycle: a company-wide intake and triage system, on the
same premise that work arriving inconsistently is a systems problem rather than a discipline problem.

## Platform beta monitoring

I tracked and tested the CRM vendor's beta features as a de facto service for the organization:
sandbox-to-production deployment, automated merge, agentic automation building, campaign associations
with custom objects, send-time optimization, publishing permissions.

This was not curiosity. Two of these shaped real project decisions. The sandbox-to-production
deployment beta became the controlled release path for a major integration go-live, and it was
adopted precisely because of the incident in
[case study 12](case-studies/12-incident-response.md) where sandbox work reached production
informally. The automated merge beta was used deliberately, with a manual review queue, during the
migration in [case study 10](case-studies/10-migration-at-scale.md).

Knowing what a platform is about to ship is part of designing on top of it. A design that works around
a limitation being removed next quarter is a design with an expiry date.

## Knowledge transfer

Every substantial piece of work shipped with a walkthrough and a written procedure. Not
documentation-as-obligation: [case study 05](case-studies/05-portal-rebuild.md) was delivered as a
handover document plus an operations playbook, written so each system's owner could take one section
and become productive without reading the rest.

The test I hold for this is whether the work survives its author. Systems that require their builder
are a liability being described as expertise.
