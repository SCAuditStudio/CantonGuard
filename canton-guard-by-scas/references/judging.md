# Judging

Every deduplicated candidate must pass these validation gates in order.

If a gate fails, reject or demote immediately. Do not loop back.

Lens output is only a lead. Start by trying to falsify it with code facts, Daml authorization rules, Canton visibility rules, and the surrounding workflow.

## Required Validation Note

Before promoting anything, write a compact note for yourself with:

- Claim: intended invariant versus alleged break, including any reporter scenario or test if supplied
- Evidence map: smallest relevant templates, choices, fields, keys, helper contracts, and file references
- Reachability: parties, `actAs`, controllers, stakeholders, observers, explicit disclosures, and key maintainers
- State trace: pre-state, submitted transaction sequence, reads, creates, archives, disclosures, and post-state
- Counter-evidence: guards, imports, tests, scripts, Canton constraints, or runtime semantics that might block the path
- Confidence: integer `0-100` and the reason the score is not higher or lower

Do not include the note verbatim in the final report unless the report format asks for that field. Use it to decide whether the candidate is a finding, a lead, or a rejection.

## Gate 0: Claim and Evidence Shape

Question: is there a specific security claim to verify, or only a vague concern?

Check:

- what security property is supposed to hold
- what transaction or workflow allegedly violates it
- which source lines or logic blocks are necessary for the claim
- whether the issue depends on production code, tests, comments, README claims, or missing code

Verdict:

- `READY` if the claim has a concrete invariant, code path, and alleged outcome
- `LEAD` if the idea is plausible but missing a complete path or source evidence
- `REJECT` if the claim is only generic lint, a feature request, or unsupported speculation

## Gate 1: Authorization and Visibility

Question: can the claimed attacker actually reach the code path under Daml and Canton visibility rules?

Check:

- is the attacker a stakeholder, controller, or valid disclosed-contract submitter?
- can they legally `fetch`, `fetchByKey`, `lookupByKey`, or `exercise` the referenced contract?
- are the required `actAs` parties available?

Verdict:

- `ALLOWS` if the attacker can truly reach the path
- `BLOCKS` if authorization or visibility makes the path impossible
- `UNCERTAIN` only if code or config is missing; treat `UNCERTAIN` as a lead, not a finding

## Gate 2: State or Privacy Consequence

Question: does the path create a real security consequence?

Acceptable consequences include:

- unauthorized archival or recreation
- unauthorized choice exercise
- replay of a one-shot action
- sensitive data disclosure through observers, choice observers, divulgence, or explicit disclosure misuse
- stuck state, lost recovery path, or broken threshold logic
- economically meaningful mismatch between checked fees or values and actual on-ledger settlement
- operator, processor, approver, or manager substitution that reroutes workflow or breaks policy assumptions
- an advertised security-critical workflow branch is unreachable or only partially wired

Reject candidates that only describe:

- stylistic issues
- expected failures
- harmless alternate modeling choices
- cosmetic API awkwardness with no workflow or security consequence
- behavior that is explicitly modeled as trusted operator discretion with no contradicted invariant

## Gate 3: Daml and Canton Semantics

Question: does the exploit still work when Daml and Canton semantics are applied correctly?

Check:

- consuming versus nonconsuming behavior
- stakeholder and witness propagation
- key maintainer authorization
- `getTime` versus caller-supplied time
- event emission or returned data versus actual contract or value movement
- caller-supplied role or config values versus authoritative template state
- domain and informee constraints
- contention and stale-contract behavior
- atomic transaction rollback on failed assertions, failed fetches, or missing authorization

If the issue depends on semantics the runtime already blocks, reject it.

## Gate 4: Before/After State Trace

Question: can you reproduce the exploit as a coherent ledger story?

Check:

- pre-state: active contracts, parties, signatories, observers, keys, relevant field values
- call sequence: submitter, `actAs`, exercised choice, fetched contracts, created contracts, archived contracts
- post-state: exact broken invariant, unauthorized visibility, replay window, stuck state, or economic mismatch
- timing or concurrency assumptions: whether the window exists after atomicity, contention, and command rejection are considered

Verdict:

- `CONFIRMED` if the before/after state proves the claimed consequence
- `LEAD` if the sequence probably exists but requires missing config, undisclosed contracts, or unreviewed off-ledger behavior
- `REJECT` if the trace cannot be completed without inventing authority, visibility, state, or timing

## Gate 5: Impact and Repeatability

Question: is the consequence material enough to report?

Prefer findings that can:

- cause or enable loss, double-claim, or broken settlement
- destroy another party's rights
- leak commercially sensitive data
- wedge core workflows repeatedly
- create governance or threshold bypass
- let one operator choose mutually exclusive settlement outcomes
- create unpaid-service, overpayment, or phantom-settlement conditions
- break an advertised production workflow that operators depend on for safety

Demote to lead if the issue is theoretically real but narrow, heavily preconditioned, or poorly evidenced.

## Promotion Rules

Promote a lead to a finding if:

- two or more lenses converge on the same root cause, and
- the code path survives all validation gates, and
- you can explain the exploit in one concrete party flow with before/after state, and
- you can name the strongest counterargument and why it does not block the path

## Rejection Rules

Reject if any of the below is true:

- the attack has no complete transaction sequence
- the attack relies on parties seeing contracts they cannot see
- the attack relies on missing authorizers the submitter cannot supply
- the issue is prevented by the type system or by default Daml authorization
- the report confuses divulgence, observers, and stakeholders
- the issue is only "missing tests" without a concrete security failure
- the alleged impact is only an emitted event, return value, or comment unless downstream production code treats it as authoritative security state
- the most credible fix would be "document the trust model" rather than changing a security-critical ledger rule

## Confidence Calibration

Use confidence as a validation score, not a severity score.

- `90-100`: complete source-backed path, direct state trace, and counter-evidence checked
- `75-89`: path is clear but minor deployment or intent assumptions remain
- `50-74`: plausible lead with missing source, config, test, or off-ledger context
- `<50`: too speculative for a finding; reject or omit from the report
