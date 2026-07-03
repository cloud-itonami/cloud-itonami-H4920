# Operator Guide

## First Deployment
1. Register carrier, lanes, vehicles and operators.
2. Import shipment and tracking history.
3. Run read-only tracking-number validation.
4. Configure exception escalation and settlement accounts.
5. Publish a dry-run settlement and audit export.

## Minimum Production Controls
- tracking-number validation before dispatch
- route and consignment integrity for every shipment
- exception escalation gate
- audit export for every settlement and claim
- backup manual dispatch and POD process

## A Day in the Life: Intake → Propose → Approve → Execute → Audit

Community Freight Transport (ISIC 4920, `cloud-itonami-4920`) runs on the same
intake/propose/approve/execute/audit loop as every itonami blueprint, but here the loop is
concrete: a rural cooperative hauler needs to move a shipment (say, a pallet of refrigerated
produce) from a collection point to a regional market on a registered lane. Walking through one
shipment, end to end:

1. **Intake.** The cooperative books the shipment through `:forms`: origin, destination,
   cargo type, weight, requested pickup window. This creates a shipment record with no
   tracking number yet — `:logistics` will not treat it as dispatch-eligible until one is
   issued.
2. **Propose.** The dispatcher (or the `:optimization` planner) proposes a lane and mode for
   the shipment — this is where multi-modal route planning happens and where
   `:decarbonization` gets acted on, by preferring the lower-emission mode/lane combination
   that still meets the pickup window. The proposal issues the shipment's tracking number.
3. **Approve.** Before the driver can leave the depot with this shipment, the
   `:freight-governor` sign-off gate has to clear: it checks the tracking number is valid, that
   the prior leg's proof-of-delivery is on record if this is a connecting lane (protects
   `:rural-access` lanes that have no fallback carrier), and that no unresolved delivery
   exception is blocking this lane. The `:dmn` decision table evaluates this; the `:bpmn`
   process holds the shipment at this step until it passes. This sign-off is single-use — it
   authorizes *this* dispatch, not a standing permit for the driver's next load.
4. **Execute.** The driver runs the lane and delivers. Proof-of-delivery is captured at
   the point of delivery (signature, photo, or scan depending on deployment) and written back
   to the shipment record — this is the artifact the next leg's Approve step will check for if
   the shipment continues on a connecting lane.
5. **Audit.** The dispatch sign-off, the POD, and (once the settlement run happens) the
   payout to the carrier are all appended to the `:audit-ledger` — immutable and exportable, so
   a claim or dispute can be traced back to the exact sign-off and POD that authorized the
   delivery. If something went wrong in transit (damaged cargo, missed window), that gets
   raised as a delivery exception and routed through the escalation gate instead of being
   silently closed out — settlement for that shipment then waits on governor approval of the
   exception's resolution.

Any deviation from this loop is exactly what the Trust Controls in `docs/business-model.md`
exist to catch: a shipment dispatched without a valid tracking number, a route cleared without
a POD chain, or a settlement posted without governor sign-off.

## Feel the Approval Gate: `itonami/freight-dispatch`

Community Freight Transport has a companion playable prototype in `network-isekai`, at
`public/games/itonami/freight-dispatch` (ADR-2607031000). It compresses the same
approve-before-execute loop from this guide into a depot round you can actually play:

- touch the **dispatch** depot to receive this round's `:freight-governor` route sign-off
  (the game's `equipped` flag — a direct stand-in for the DMN-evaluated approval above),
- then clear a **route** job — clearing one while signed off despawns it and scores a point,
  the sign-off is consumed (single-use, matching rule 3 above),
- a rarer **priority-route** bonus job is worth 3x score under the same sign-off rule — no
  shortcut around the governor for high-value freight,
- clearing a route **without** checking in first is an unauthorized run: it costs a life
  instead of a point, the same failure mode the audit ledger is built to catch and the
  minimum production controls above are built to prevent.

It is not a substitute for the controls in this guide, but it is the fastest way for a new
operator (or a reviewer) to feel, hands-on, why the sign-off gate exists before touching a real
deployment.

## Certification
Certified operators must prove dispatch integrity, evidence-backed POD and
human review for settlement-affecting actions.
