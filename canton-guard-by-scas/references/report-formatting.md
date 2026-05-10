# Report Formatting

Produce one final report. Do not print raw lens outputs.

## Header

Start with:

- `# Daml Security Report`
- `Scope:` one line describing audited files or modules
- `Focus:` one line describing the main attack surface

If useful, add `Context:` with SDK version, package layout, or Canton deployment assumptions taken from `daml.yaml` or config.

## Findings Section

List findings first, ordered by severity, then confidence.

Use this structure for every accepted finding:

### [SEVERITY-ID] Title

- Severity: `Critical`, `High`, `Medium`, or `Low`
- Confidence: integer `0-100`
- Area: short label such as `authorization`, `privacy`, `contract-key`, `state-machine`, `time`, `quorum`
- Group key: `Template | choice-or-operation | bug-class`
- Location: file path and the smallest useful template or choice name
- Claim check: intended invariant versus the actual ledger behavior
- Preconditions: external setup, active contracts, parties, and required visibility
- Attack path: concrete transaction sequence, including submitter and `actAs` parties
- State change: before/after state, including creates, archives, disclosures, or value movement
- Impact: concrete user or protocol consequence
- Why it works: the exact Daml or Canton rule the code violates or misunderstands
- Counter-evidence checked: strongest guard, semantic rule, test, or workflow assumption that was considered and why it does not block the issue
- Minimal fix: the shortest safe remediation direction
- Suggested tests: `submitMustFail`, boundary-time, negative authorization, replay, or invariant tests as appropriate

Keep each finding compact. Prefer evidence over narrative. Do not include a finding if you cannot fill in the attack path and state change without guessing.

## Leads Section

If useful, add:

## Leads

Use this only for plausible issues that did not clear the full judging bar.

For each lead:

- Title
- Why it might matter
- Failed validation gate or missing evidence
- What evidence is still missing

## Coverage Notes

End with:

## Coverage Notes

Include:

- major modules reviewed
- major classes of risk considered
- anything not verified because code or config was missing

## Severity Guidance

Use this mapping:

- `Critical`: systemic loss of asset integrity, key uniqueness, or privacy guarantees across core flows
- `High`: unauthorized archival, repeatable claim or execution, threshold bypass, major privacy leak, or permanent loss of workflow control
- `Medium`: meaningful workflow break, denial of service, time bypass, or authority confusion with bounded blast radius
- `Low`: real but narrow issue, ambiguous attestation model, or overexposure with limited practical harm

Do not inflate severity because the project is finance-adjacent.
