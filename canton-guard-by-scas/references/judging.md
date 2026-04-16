# Judging

Every deduplicated candidate must pass these four gates in order.

If a gate fails, reject or demote immediately. Do not loop back.

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

Reject candidates that only describe:

- stylistic issues
- expected failures
- harmless alternate modeling choices

## Gate 3: Daml and Canton Semantics

Question: does the exploit still work when Daml and Canton semantics are applied correctly?

Check:

- consuming versus nonconsuming behavior
- stakeholder and witness propagation
- key maintainer authorization
- `getTime` versus caller-supplied time
- domain and informee constraints
- contention and stale-contract behavior

If the issue depends on semantics the runtime already blocks, reject it.

## Gate 4: Impact and Repeatability

Question: is the consequence material enough to report?

Prefer findings that can:

- cause or enable loss, double-claim, or broken settlement
- destroy another party's rights
- leak commercially sensitive data
- wedge core workflows repeatedly
- create governance or threshold bypass

Demote to lead if the issue is theoretically real but narrow, heavily preconditioned, or poorly evidenced.

## Promotion Rules

Promote a lead to a finding if:

- two or more lenses converge on the same root cause, and
- the code path survives all four gates, and
- you can explain the exploit in one concrete party flow

## Rejection Rules

Reject if any of the below is true:

- the attack relies on parties seeing contracts they cannot see
- the attack relies on missing authorizers the submitter cannot supply
- the issue is prevented by the type system or by default Daml authorization
- the report confuses divulgence, observers, and stakeholders
- the issue is only "missing tests" without a concrete security failure
