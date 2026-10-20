# Two simulated operational journeys

These fictional examples illustrate the proposed design. They are not execution transcripts or measured results. Role names describe business responsibility, not separate AI agents. Company rules determine authority in every case.

## A. A coverage coordinator receives authorised absences

**Situation.** Several employees will be absent during a period with required on-call coverage. People records identify qualification levels; the schedule defines coverage; the absence system defines availability. External support has its own availability, onboarding requirements and rates.

**Decision.** ORBIT checks the requester's delegation and the requested period. It reads the relevant records, flags missing information and identifies uncovered intervals. Employee L1-01 appears available but fails the rest requirement. Moving L2-03 would leave another required interval uncovered. A compliant internal reassignment and an external engagement remain candidates.

ORBIT compares their operational consequences and cost. The lowest-cost candidate need not be valid, and the design does not claim a globally optimal answer. If no option satisfies the current constraints, it identifies the specific decision or information needed from the coordinator.

**Controlled action.** Before external spending, the gate atomically reserves funds in the authoritative controlled ledger. Before changing a shift, it checks the expected record version where supported. A stable operation identity links retries to the original request.

**Failure branch.** The external provider times out after receiving the engagement. ORBIT queries its state; it does not submit a new purchase or release the reservation on the assumption that nothing happened. If a concurrent schedule change invalidates the plan, dependent effects stop while confirmed commitments are reconciled. Completion or compensation must preserve coverage and remain authorised.

**Expected evidence.** An updated schedule satisfies affected coverage, rest, qualification and availability rules. The cost record matches confirmed engagements and outstanding reservations. The report identifies verified effects and any unresolved state.

Illustrative closure: “Coverage verified for the requested period. Applied changes and remaining budget are recorded. No affected interval remains uncovered.” This statement is permitted only if those observations exist.

## B. An incident operator investigates a critical outage

**Situation.** A monitor reports an unavailable service and an incident is classified P1: a critical incident. Recent deployment activity is relevant evidence, alongside other changes that may be unrelated.

**Investigation.** ORBIT compares monitoring, configuration and logs. In this example, routing sends traffic to port **8081**, while the application listens on **8080**. The deployment's timing is a clue; the configuration mismatch and observed failures support the hypothesis.

**Decision.** Three options are considered: correct the destination port, move traffic to the previous version or revert the deployment. A targeted correction preserves the rest of the rollout when evidence supports it. A known rollback may be safer when uncertainty remains. The incident's urgency does not grant a write capability.

**Controlled action.** If remediation is authorised, the gate checks the current configuration version and applies the permitted change. Otherwise ORBIT provides the diagnosis and requests the required authorised action. Connectors must support and validate the requested operation; describing the capability does not implement it.

**Failure branch.** The configuration API returns success, but the service remains unavailable. ORBIT keeps the incident open, records the mismatch and revisits its hypothesis. It reverts or compensates only through an authorised operation with understood effects.

**Expected evidence.** Service health, external access, error behaviour and relevant routing state satisfy predefined criteria during an observation window. The incident record reflects the verified operational state.

Illustrative closure: “Recovery verified independently. The change was limited to the affected routing configuration; before-and-after observations are attached.” A P2 or another incident type would select its own company-defined response rules without changing the underlying authority model.
