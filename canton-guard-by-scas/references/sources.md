# Sources

Use primary sources first. Use secondary material to generate hypotheses, then verify against the primary sources before promoting a finding.

## Primary Sources

### Core Daml semantics

- Templates: https://docs.daml.com/daml/reference/templates.html
  Use for `signatory`, `observer`, and template-level authorization semantics.

- Choices: https://docs.daml.com/daml/reference/choices.html
  Use for controller rules and consuming versus nonconsuming behavior.

- Updates: https://docs.daml.com/daml/reference/updates.html
  Use for `create`, `archive`, `exercise`, `fetch`, and key operations.

- Contract keys: https://docs.daml.com/daml/reference/contract-keys.html
  Use for `maintainer`, `visibleByKey`, `lookupByKey`, and `fetchByKey` behavior.

- Ledger privacy: https://docs.daml.com/concepts/ledger-model/ledger-privacy.html
  Use for stakeholders, informees, witnesses, and divulgence.

### Canton-specific behavior

- Security architecture: https://docs.daml.com/canton/architecture/security.html
  Use for Canton trust assumptions and operational security boundaries.

- Explicit contract disclosure: https://docs.daml.com/app-dev/explicit-contract-disclosure.html
  Use when mixed-party transactions depend on off-ledger disclosure.

- Ledger API reference: https://docs.daml.com/app-dev/grpc/proto-docs.html
  Use when evaluating `disclosed_contracts`, visibility checks, and command-submission behavior.

- Error codes: https://docs.daml.com/canton/reference/error_codes.html
  Use as evidence for likely failure modes and operational consequences.

### Workflow and modeling patterns

- Propose and Accept: https://docs.daml.com/daml/patterns/propose-accept.html
- Multiple Party Agreement: https://docs.daml.com/daml/patterns/multiparty-agreement.html
- Authorization: https://docs.daml.com/daml/patterns/authorization.html
- Delegation: https://docs.daml.com/daml/patterns/delegation.html
- Locking: https://docs.daml.com/daml/patterns/locking.html

Use these to judge whether the code is following or violating known-safe Daml patterns.

## Secondary Sources

### User-provided material

- Common vulnerabilities article:
  https://github.com/SCAuditStudio/SCASArticles/blob/main/DamlSmartContractVulnerabilities.md

- Canton patterns reference implementation:
  https://github.com/JohnLilic/canton-patterns

- Canton patterns audit notes:
  https://github.com/JohnLilic/canton-patterns/blob/main/AUDIT.md

Use these to seed hypotheses, examples, and pattern-specific checks. Verify all substantive claims against the primary sources above before escalating them into findings.

## Source Discipline

- Cite primary sources in final explanations whenever possible.
- Do not treat blog language or audit commentary as authoritative if it conflicts with official docs.
- If you make an inference from the docs, say that it is an inference.
