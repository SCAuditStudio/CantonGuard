# Daml and Canton Attack Vectors

Use this file to generate candidate issues. Do not report every bullet by default. Trace the concrete party flow first.

## Core Mindset

Daml removes many low-level EVM bugs. The audit question is usually:

- who can create this right?
- who can archive it?
- who can exercise which choice?
- who can see the data?
- who can replay the action?
- who can recover if the workflow stalls?
- where does value actually move, and what is only recorded in an event?
- which stored configuration fields are truly enforced?
- is the advertised workflow actually reachable from on-ledger entry points?

## 1. Stakeholder and Signatory Design

Look for contracts where a protected counterparty is only an observer.

Questions:

- Can one party archive the contract unilaterally?
- Does a workflow need a propose-accept or pending-agreement wrapper before a multi-party contract is created?
- Is a receipt or result contract signed only by the initiator when multiple parties rely on it as proof?

Typical impact:

- unilateral archival
- off-ledger obligations without on-ledger protection
- one party can rewrite or erase another party's state

## 2. Controller and Delegation Confusion

Look for flexible controllers, delegation wrappers, intermediate authority objects, or helper choices.

Questions:

- Does the controller match the intended principal?
- Is a delegated actor constrained to a narrow action, or can they launder broader authority?
- Can a helper contract be used to act on behalf of a party that never explicitly approved this exact action?

Typical impact:

- privilege escalation
- authority laundering
- exercising a legitimate choice for an illegitimate business purpose

## 3. Consuming versus Nonconsuming Choices

Look for any choice that transfers value, finalizes status, issues receipts, claims rewards, redeems vouchers, or settles obligations.

Questions:

- Should this choice be one-shot?
- Does replay create duplicated payments, claims, or approvals?
- Does the contract stay active after an action that sounds terminal?

Typical impact:

- replayable redemption
- repeatable settlement
- double-claim
- duplicated governance actions

## 4. Missing Preconditions and Weak Invariants

Check every `ensure`, `assert`, and `assertMsg` boundary.

Questions:

- Are amounts positive and bounded?
- Are lists deduplicated where thresholds depend on list cardinality?
- Can `claimed`, `released`, `yesVotes`, `approvals`, or other cumulative counters exceed the intended limit?
- Do deadline or cliff relationships hold?

Typical impact:

- quorum collapse
- negative or zero-value abuse
- over-claim or over-release
- malformed state that later choices cannot handle safely

## 5. Stored Configuration versus Caller Input

Look for choices that accept role-like or config-like values even though the template already stores them.

Questions:

- Does the choice accept `operator`, `processor`, `provider`, `manager`, `approver`, `owner`, or similar from the caller?
- Is the supplied value checked against the template's stored field?
- Can the caller reroute workflow through an unintended operator, processor, service-provider, or approver?
- Are redundant parameters likely to create a second, non-authoritative source of truth?

Typical impact:

- operator or service-role substitution
- authorization laundering through caller-supplied config
- operational misconfiguration because integrators trust the wrong field

## 6. Economic Settlement and Event-Only Logic

Look for code that talks about payment, fees, or value transfer but only emits data.

Questions:

- Does the code enforce a fee or amount without moving any on-ledger asset or right?
- Is a `value` copied into an event while no settlement occurs?
- Does `>=` allow overpayment, and if so, where is the refund path?
- Are comments, README text, or UI labels implying a stronger accounting guarantee than the ledger code provides?

Typical impact:

- unpaid service providers or processors
- phantom fee enforcement
- silent user overpayment
- broken incentive model

## 7. Privacy, Witnesses, and Divulgence

Mixed-party transactions deserve special attention.

Questions:

- Are observers broader than necessary?
- Does exercising a choice force disclosure of a private contract to a party that only needed to trigger the workflow?
- Is the design relying on accidental divulgence rather than explicit modeling?
- Would explicit contract disclosure be safer than `fetch` inside a mixed-party transaction?

Typical impact:

- trade detail leakage
- confidential pricing or position disclosure
- silent expansion of who can inspect sensitive state

## 8. Contract Keys and Visibility by Key

Audit every keyed template and every `lookupByKey` or `fetchByKey`.

Questions:

- Is the maintainer derived correctly and guaranteed non-empty?
- Is the maintainer a signatory?
- Does the caller actually satisfy `visibleByKey`?
- Does the code treat `None` from `lookupByKey` as "does not exist" when the real issue is "not visible to this caller"?
- Are key-based uniqueness assumptions still valid across participants or with explicit disclosure?

Typical impact:

- broken uniqueness assumptions
- invisible contracts mistaken for absent contracts
- logic branches that mint duplicates or bypass authorization because lookups fail closed

## 9. Time and External Inputs

Any business-critical time gate should be traced carefully.

Questions:

- Does the choice accept `currentTime`, `now`, or `deadline` from the caller instead of using `getTime`?
- Can a party fast-forward claims, unlocks, cancellations, or revocations?
- Are before/after deadline checks symmetric across execution and cancellation paths?

Typical impact:

- premature vesting or release
- backdated revocation
- cancellation after outcome is known
- refund or settlement race conditions

## 10. State Machine Completeness

Look for proposal, funded, disputed, released, revoked, expired, cancelled, or tallied states.

Questions:

- Are all expected terminal and recovery transitions present?
- Can assets or rights become stuck with no authorized exit?
- Can a contract enter a state that no controller can progress?
- Is there a recovery path after timeout or dispute?
- Are mutually exclusive terminal outcomes tied to verifiable state, proof, or timeout?
- Can one operator decide between release, liquidation, refund, or seizure after a counterparty has already committed value?

Typical impact:

- stuck funds
- permanent workflow denial of service
- business dead ends
- unilateral settlement outcome selection
- premature liquidation or seizure

## 11. Quorum and Set Semantics

Threshold systems often fail because lists are treated as sets without enforcement.

Questions:

- Are signers, voters, approvers, maintainers, or delegates unique?
- Can one party appear multiple times to satisfy a threshold?
- Is the proposer allowed to execute without being part of the signing threshold, and is that intentional?

Typical impact:

- fake M-of-N security
- duplicate approvals
- governance bypass

## 12. Canton Operational Assumptions

These are often not code bugs by themselves, but they matter for real exploitability and denial of service.

Questions:

- Does the workflow assume all informees are active on a common domain?
- Does it assume a submitter can act via a participant that may not host them?
- Does it assume contracts are on a known domain during transfer-heavy flows?
- Does it ignore contention or stale-contract races in concurrent submissions?

Relevant error-code patterns:

- `DAML_AUTHORIZATION_ERROR`
- `CONTRACT_NOT_FOUND`
- `PERMISSION_DENIED`
- `INFORMEES_NOT_ACTIVE`
- `SUBMITTER_CANNOT_ACT_VIA_PARTICIPANT`
- `UNKNOWN_CONTRACT_DOMAINS`

Typical impact:

- workflow-level denial of service
- false assumptions in off-ledger orchestration
- race-prone settlement or transfer handling

## 13. Feature Reachability and Partial Wiring

Some real bugs are missing or orphaned workflow branches rather than broken single choices.

Questions:

- Do sibling modules, comments, README text, or callers advertise a workflow that no reachable choice can instantiate?
- Is only one branch of a family of proposals, agreements, or safety wrappers wired into the public entry points?
- Does an "emergency", "restricted", or "safe" flow exist only as dead-end code while production callers use a weaker path?

Typical impact:

- business-critical workflow unavailable in production
- false assumptions by integrators and operators
- security properties present in code comments but absent in live flows

## 14. Pattern-Specific Checks

### Access control

- self-grant or self-approval edge cases
- admin can silently rewrite roles without counterparty acknowledgement
- observer/signatory mismatch on role receipts
- caller-supplied operator, processor, manager, or approver not bound to stored configuration

### Escrow and safekeeping

- no refund path after timeout
- release or dispute reachable before deposit
- beneficiary or payer can force an unintended terminal path
- one operator can choose between refund and seizure without proof or timeout

### Multisig

- duplicate signers
- double sign replay
- execute or cancel authorization too broad
- threshold not bounded by unique signer count

### Vesting

- caller-supplied time
- claim amount exceeds vested amount
- revoke path backdates unvested amount
- beneficiary or grantor can trap the other party in an unresolved state

### Timelock

- cancel after unlock
- execute before unlock
- receipt contract proves less than users think it proves

### Voting

- duplicate voters
- late cancellation
- tally before deadline
- tie or abstain rules undocumented and security-relevant

### Coin, token, or asset workflows

- issuer can archive live assets at will
- transfer proposal bypasses recipient acceptance
- locking workflow does not actually lock the risky choices
- a fee check creates an event but no actual payment
- overpayment is accepted without refund or explicit semantics

## 15. Tests That Should Usually Exist

Strong Daml security reviews usually expect:

- `submitMustFail` for unauthorized parties
- replay tests for any one-shot choice
- boundary-time tests at `== deadline`, `< deadline`, and `> deadline`
- duplicate-list tests for multisig, voting, or delegates
- negative-value, zero-value, and over-claim tests
- recovery-path tests for expiry, refund, or dispute flows
- config-binding tests that caller-supplied operator or service role matches stored configuration
- exact-fee and overpayment tests where payment semantics matter
- reachability tests for every advertised workflow branch, agreement type, or safety wrapper
- exclusivity tests for mutually exclusive settlement outcomes
