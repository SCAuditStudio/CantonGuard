# Business Logic Checks

Use this checklist to catch workflow and economic bugs that are easy to miss in otherwise type-safe Daml code.

## 1. Claimed Workflow versus Reachable Workflow

Cross-check README text, sibling modules, scripts, and UI handlers against the actual on-ledger entry points.

Questions:

- Is there a template or proposal type that no reachable choice ever creates?
- Does the advertised "safe", "restricted", or "reviewed" flow only exist as a module, while callers can only create the simpler path?
- Does off-ledger code assume a feature branch that the ledger code never wires in?

High-signal outcomes:

- security-critical workflow branch is unreachable
- partially implemented feature creates false operator assumptions
- product promises rely on a path that does not exist on-ledger

## 2. Stored Configuration versus Caller Input

Treat role-like and config-like choice arguments as suspicious when the template already stores the same concept.

Questions:

- Does the choice accept `operator`, `processor`, `provider`, `manager`, `approver`, `owner`, or similar?
- Is that argument checked against the template field, or can the caller substitute an arbitrary party?
- Are redundant parameters likely to mislead integrators about which value is authoritative?

High-signal outcomes:

- operator or service-role substitution
- policy bypass through caller-selected roles
- operational misconfiguration from shadowed state

## 3. Event-Only Accounting and Phantom Settlement

Do not confuse emitted events or returned records with actual settlement.

Questions:

- Is a fee or value merely checked and then copied into an event?
- Does the ledger actually create, archive, or transfer any contract that represents the claimed payment?
- Is a service described as "paid" when the code only enforces `value >= fee` but never settles it?

High-signal outcomes:

- unpaid service-provider or processor flow
- broken incentive model
- false assurance that fees are enforced

## 4. Exactness, Overpayment, and Refund Semantics

Permissive inequalities can hide real value-flow bugs.

Questions:

- Should the code require `==` rather than `>=`?
- If overpayment is allowed, where is the explicit refund or accounting path?
- Is the overpayment intentionally retained, or simply ignored by the ledger model?

High-signal outcomes:

- avoidable user overpayment
- silent protocol windfall
- ambiguous fee semantics that downstream callers mis-handle

## 5. Outcome Control after Counterparty Commitment

Once another party has deposited value or accepted risk, terminal outcomes should rarely remain under one operator's sole control.

Questions:

- Can one party choose between unlock, refund, liquidation, cancel, or seize without counterparty re-authorization?
- Is an emergency path gated by verifiable state, proof, timeout, or dispute state, or only by operator intent?
- Are execution flags written but not consulted by neighboring terminal choices?

High-signal outcomes:

- unilateral settlement outcome selection
- premature liquidation
- double-settlement or broken workflow-settlement invariants
