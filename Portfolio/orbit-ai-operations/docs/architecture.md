# Architecture: flexible investigation, controlled execution

This document describes the proposed responsibilities and contracts behind the ORBIT portfolio. Examples are illustrative. The design separates four questions: what happened, what should change, who permits the change and how the outcome is checked.

## Responsibilities

| Component | Responsibility | Boundary |
|---|---|---|
| Decision core | Gather facts, form hypotheses, compare options and propose actions | Cannot grant permissions or change company policy |
| Capability registry | Describe available operations and their contracts | Contains no scenario-specific solution |
| Connectors | Translate requests to provider APIs and preserve provenance | Do not quietly decide the business outcome |
| Policy evaluator | Check authority and operational constraints | Runs outside the language model |
| Execution gate | Revalidate conditions and dispatch authorised mutations | Enforces version, budget and operation identity |
| Verifier | Read resulting state and evaluate success criteria | Does not trust a mutation acknowledgement alone |
| Audit record | Preserve request, authority, actions and observations | Records observable facts and concise rationale |

These responsibilities may share infrastructure. They are separated logically so that a model's explanation cannot bypass enforcement.

## Familiar enterprise integrations

The prototype used Mattermost as a no-cost chat option. It is an interchangeable interface, not a dependency of the decision core. A Slack or Microsoft Teams adapter could receive requests and return progress, approval requests and outcomes; the organisation's authenticated identity and approval policy would still determine authority. Chat content alone does not grant permission.

For HR, a Workday adapter—or an adapter for another HR platform with a suitable API—could supply the permitted people, absence and organisational data required by the workflow. Writes would require separately authorised operations. Workday offers REST and SOAP web services; the choice depends on the operation exposed and enabled in the customer's environment.

| Integration boundary | Proposed adapter | Validation needed |
|---|---|---|
| Chat requests and updates | Slack or Microsoft Teams | Identity mapping, permitted channels, delivery and retry behaviour |
| HR context and authorised changes | Workday or another HR platform API | Available operations, field mapping, access scopes and authoritative records |

These are extension paths, not integrations demonstrated in this package. Replacing an adapter preserves the design's policy and verification responsibilities; it does not imply identical provider capabilities.

API references: [Slack APIs](https://docs.slack.dev/apis/), [Microsoft Teams messaging via Microsoft Graph](https://learn.microsoft.com/graph/api/chatmessage-post), [Workday integration APIs](https://www.workday.com/content/dam/web/en-us/documents/datasheets/datasheet-workday-cloud-connect-platform.pdf).

## Discovering a capability

The core receives a versioned manifest of authorised operations. Each describes its inputs, outputs, effects, errors and verification method. Administrative reset, fault injection and evaluation reference answers remain outside normal discovery.

An illustrative contract fragment:

```yaml
capability: coverage.assignment.replace
version: 1
inputs:
  assignment_id: string
  replacement_person_id: string
  expected_version: string
  operation_id: string
  plan_reference: string
effect: Replace one future assignment
required_scope: coverage.write
preconditions:
  - Current delegation covers the resource and period
  - Replacement satisfies availability, rest and qualification rules
  - Expected version still matches
verification:
  - Read assignment from its authoritative system
  - Recheck affected coverage and conflicts
errors: [FORBIDDEN, CONFLICT, UNAVAILABLE, UNCERTAIN]
recovery: Re-read before any retry; compensation requires separate authority
```

This contract requires a connector and validation against the actual provider. Listing an operation does not make it available. Stable identifiers help correlate retries; duplicate prevention also depends on durable enforcement by the gate and destination.

## Policy outside the model

Authority identifies the actor, action, resource, period and applicable limits. Approval for an exception is bound to the relevant proposal or plan version; materially changed effects require revalidation. Company policy distinguishes approved discretion from mandatory prohibitions.

The AI may recommend a staffing option. Deterministic checks must still reject an unavailable person, an unauthorised purchase or a forbidden change. Incident priority selects applicable response rules and urgency; it does not extend permissions.

Missing or contradictory information remains explicit. If an unknown leave balance affects one assignment, the system blocks dependent effects. Independent work proceeds only when separation has been established; the system must not assume every plan can be split safely.

## Spending without racing the budget

A balance read followed by a purchase is insufficient: two requests may both see the same available funds.

The proposed gate uses an **atomic reservation** against a versioned budget ledger before dispatching an external commitment. The reservation and stable operation identity are recorded together. It counts outstanding reservations as well as confirmed commitments.

After dispatch, the gate reconciles the provider's state: confirm the commitment, release a definitively rejected reservation or retain an uncertain reservation pending investigation. A timeout is not evidence that money was uncommitted. Cancelling a reservation does not cancel an external engagement.

This works only when relevant writers use the ledger or reconcile through an authoritative source with equivalent guarantees. If spending can bypass that boundary, the system cannot promise global budget enforcement; it restricts automated commitments until the authority and outstanding spending are reconciled.

## Concurrent changes

When the destination API supports conditional writes, the connector sends the version it read. A mismatch rejects the dependent operation and triggers a refresh.

Without that support, a serial execution gate can coordinate participating writers using durable state and a lease or fencing mechanism appropriate to the destination. It cannot stop an unrelated administrator writing directly. Where exclusion cannot be enforced and the impact is unsafe, the gate fails closed and requests a controlled maintenance or human path. An immediate pre-write read alone does not remove the race.

## Partial effects and safe recovery

There is no assumed transaction across people, scheduling, vendor and messaging systems. The record tracks planned, dispatched, confirmed, uncertain and verified effects separately.

After a partial failure, the system checks what actually changed. It completes remaining actions only if current constraints permit. It compensates confirmed effects only through an authorised operation whose consequences have been checked. Otherwise it stops new dependent effects, preserves the confirmed state and reports the unresolved risk. Restarting the whole plan must not duplicate an engagement or undo another person's valid change.

## Verification and bounded operation

An action acknowledgement means the provider accepted a request. Verification means an authoritative observation supports the intended effect and the relevant constraints. The verifier uses bounded retries and observation windows for eventual consistency, and reports uncertainty if evidence remains insufficient.

For coverage, it checks affected intervals and conflicts. For recovery, it checks the service, access path and relevant configuration rather than ticket state alone. Time, tool attempts, cost and cycles without progress are bounded by configuration.

The audit retains operation identity, source versions, authority, concise decision rationale, receipts and observations. Sensitive content is minimised. Hash-linked records can help reveal alteration, but require independent anchoring to detect truncation reliably.

## Evaluation

Use labelled valid and invalid cases, changed policies, unavailable sources, concurrent writes, lost responses and permission-expanding input. Measure both unsafe acceptance and **false denial** of valid actions. A high rejection rate alone is not evidence of safety.

Assess verified objectives, duplicate or unresolved effects, human intervention and reviewer reconstruction of the record. Compare time against a manual baseline and separate active work from waiting. These are proposed evaluation methods, not measured outcomes.

Docker Compose sandbox material and Cloudflare Workers/D1 designs informed the original project. They are implementation options with different responsibilities; their presence does not demonstrate a single integrated runtime. REST or MCP transport still requires validated contracts and scoped credentials for each connected system.
