# Vulnerability Checklist

Use this checklist to guarantee explicit coverage of the vulnerability classes named in the user-provided DAML best-practices README.

Do not report every item by default. For each item, make a conscious pass and decide whether it is present, absent, or not applicable in the audited workflow.

## Common Pitfalls

- `Overly permissive choice controllers`
  Check whether the controller comes from a caller-controlled argument or other non-authoritative source instead of a trusted template field.

- `Missing signatory authorization on contract creation`
  Check whether a choice creates a contract that names another party as signatory, debtor, issuer, or obligor without a fresh acceptance step from that party.

- `Unintended observer exposure (privacy leakage)`
  Check whether observers are broader than necessary or whether a summary contract should exist instead of exposing the full payload.

- `Unsafe contract key uniqueness assumptions`
  Check whether `lookupByKey = None` is treated as proof of global absence or uniqueness without a consuming serialization path.

- `Missing ensure precondition checks`
  Check whether contract creation admits negative amounts, empty participant sets, inverted time ranges, self-dealing parties, or other malformed states.

- `Arithmetic logic errors in governance or financial calculations`
  Check for divide-before-multiply logic, premature truncation, bad rounding order, and threshold math that does not match the intended business invariant.

- `Division by zero in Decimal calculations`
  Check whether counts, weights, pool sizes, supplies, or other divisors can become zero during normal or adversarial execution.

- `Time-of-check to time-of-use with getTime`
  Check not only caller-supplied time, but also whether ledger-time skew near a boundary makes a `getTime` gate weaker than the workflow assumes.

- `Consuming choice used where nonconsuming is intended`
  Check whether query, preview, lookup, comment, or read-only choices archive state accidentally because they were left consuming.

- `Unchecked archive leading to asset destruction`
  Check whether a choice archives a right, asset, or position without creating the intended successor contract in the same transaction.

- `Unrestricted delegation chains`
  Check whether a delegate can sub-delegate recursively without depth bounds, allowlists, or narrow action scoping.

- `Missing error handling on lookupByKey`
  Check whether absent or concurrently archived keyed contracts cause workflow-wide aborts through unsafe unwrapping or unchecked `fetchByKey`.

## Design Pattern Attacks

- `Authority smuggling via flexible controllers`
  Check whether a lower-privilege controller can exercise a choice body that creates, archives, or settles using a higher-privilege signatory's inherited authority.

- `Contract key collision abuse`
  Check whether a maintainer can race duplicate keyed creations through concurrent submissions or nonconsuming service entry points.

- `Signatory bait: tricking parties into becoming signatories`
  Check whether a party is asked to become signatory on a template whose sibling choices let the other party use that authority against them later.

- `Separation of duties violation via controller reassignment`
  Check whether the proposer, operator, or manager can pick or rewrite the approver, reviewer, or delegate to a colluding or arbitrary party.

- `Stale contract reference exploitation`
  Check whether long-lived stored `ContractId` references can go stale between selection and use, especially around settlement, escrow, and transfer flows.

- `Upgrade mechanism hijacking`
  Check whether archive-and-recreate upgrade flows let one party change fees, rights, or material terms without renewed counterparty consent.

- `Denial-of-service via choice spam`
  Check whether broad or nonconsuming entry points can flood triggers, comments, notifications, or auxiliary contract creation.

- `Privacy inference via transaction timing`
  Check whether notification timing, trigger cadence, or correlated auxiliary contracts reveal private business activity even when payload visibility is limited.

## Coverage Rule

Before finishing the scan, confirm that each named item above was considered at least once. If an item is irrelevant, treat it as checked and move on instead of forcing a finding.
