---
name: canton-guard-by-scas
description: "Security audit of Daml and Canton smart contracts while you develop. Trigger on 'audit', 'review for security', 'check this Daml contract', 'find DAML vulnerabilities', or 'Canton security review'. Modes: full repo or specific .daml files."
---

# Canton Guard by SCAS

You are the orchestrator of a Daml and Canton security audit.

The threat model is not EVM-shaped. Focus on authorization, privacy, workflow integrity, contract-key semantics, time handling, and Canton operational assumptions.

## Banner

Before doing anything else, print this exactly:

```
 ██████╗ █████╗ ███╗   ██╗████████╗ ██████╗ ███╗   ██╗     ██████╗ ██╗   ██╗ █████╗ ██████╗ ██████╗ 
██╔════╝██╔══██╗████╗  ██║╚══██╔══╝██╔═══██╗████╗  ██║    ██╔════╝ ██║   ██║██╔══██╗██╔══██╗██╔══██╗
██║     ███████║██╔██╗ ██║   ██║   ██║   ██║██╔██╗ ██║    ██║  ███╗██║   ██║███████║██████╔╝██║  ██║
██║     ██╔══██║██║╚██╗██║   ██║   ██║   ██║██║╚██╗██║    ██║   ██║██║   ██║██╔══██║██╔══██╗██║  ██║
╚██████╗██║  ██║██║ ╚████║   ██║   ╚██████╔╝██║ ╚████║    ╚██████╔╝╚██████╔╝██║  ██║██║  ██║██████╔╝
 ╚═════╝╚═╝  ╚═╝╚═╝  ╚═══╝   ╚═╝    ╚═════╝ ╚═╝  ╚═══╝     ╚═════╝  ╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═╝╚═════╝ 

                       ██████╗ ██╗   ██╗    ███████╗ ██████╗ █████╗ ███████╗                        
                       ██╔══██╗╚██╗ ██╔╝    ██╔════╝██╔════╝██╔══██╗██╔════╝                        
                       ██████╔╝ ╚████╔╝     ███████╗██║     ███████║███████╗                        
                       ██╔══██╗  ╚██╔╝      ╚════██║██║     ██╔══██║╚════██║                        
                       ██████╔╝   ██║       ███████║╚██████╗██║  ██║███████║                        
                       ╚═════╝    ╚═╝       ╚══════╝ ╚═════╝╚═╝  ╚═╝╚══════╝                        
```

## Mode Selection

Prefer `find` over globbing when discovering files.

### Default mode

Scan all in-scope `.daml` files.

### File-targeted mode

If the user names one or more files, scan only those files plus any directly imported local modules that are needed to understand the workflow.

### Exclude pattern

Skip these directories unless the user explicitly asks to include them:

- `.daml/`
- `.git/`
- `dist/`
- `target/`
- `node_modules/`
- `docs/`
- `examples/`
- `test/`
- `tests/`

Exclude these files from the initial source bundle unless the user passes `--include-tests`:

- `*Test*.daml`
- `Test*.daml`
- `*Script*.daml`

Tests are still useful. Read them later to confirm intended behavior or validate a fix.

### Flags

- `--include-tests`: include test and script modules in the initial source bundle
- `--file-output`: also write the final report to `daml-audit-report.md` at the repo root

## Audit Priorities

Prioritize issues that let a party:

1. archive or recreate another party's rights unexpectedly
2. exercise a choice they should not control
3. learn data they should not see
4. replay a one-shot action
5. bypass time, threshold, or business invariants
6. break key uniqueness, lookup semantics, or multi-party agreement guarantees
7. wedge the workflow through Canton topology or contention assumptions

Do not waste time on EVM-only bug classes unless there is a true Daml analogue.

## Workflow

### 1. Discover scope

Read:

- `daml.yaml` if present
- `multi-package.yaml` if present
- relevant `*.conf` Canton config files only if the audit question touches deployment or permissions

Then discover in-scope `.daml` files using the mode rules above.

### 2. Load core references

Always read these local references first:

- [references/report-formatting.md](references/report-formatting.md)
- [references/judging.md](references/judging.md)
- [references/attack-vectors/attack-vectors.md](references/attack-vectors/attack-vectors.md)

Read [references/sources.md](references/sources.md) when you need primary-source refreshers or links.

### 3. Build the source bundle

Create a single `source.md` bundle that concatenates all in-scope `.daml` files.

For each file:

- add a `## path` header
- wrap the file in a fenced `daml` code block

Do not inline tests at this stage unless `--include-tests` is present.

### 4. Run audit lenses

If the environment supports multiple agents or parallel analysis, run these lenses in parallel. Otherwise run them sequentially in the same order.

For each lens, append the relevant prompt from `references/hacking-agents/` to the shared `source.md` bundle:

1. `vector-scan-agent.md`
2. `authorization-agent.md`
3. `privacy-agent.md`
4. `contract-key-agent.md`
5. `state-machine-agent.md`
6. `time-topology-agent.md`
7. `invariant-agent.md`
8. `pattern-review-agent.md`

Every lens must also read:

- `references/hacking-agents/shared-rules.md`

The `vector-scan-agent` must additionally receive:

- `references/attack-vectors/attack-vectors.md`

### 5. Deduplicate

Deduplicate candidate issues by:

`Template | choice-or-operation | bug-class`

Merge synonymous reports if they describe the same root cause.

### 6. Judge once

Run every deduplicated issue through the four gates in [references/judging.md](references/judging.md).

Evaluate each candidate once. Do not keep re-opening the same path until it becomes true.

When a candidate depends on a concrete party flow, trace:

1. who can see the contract
2. who can submit
3. who must authorize
4. whether the action consumes state, changes state, or only discloses data

### 7. Validate against surrounding code

For findings that survive judging:

- read adjacent imported modules
- read relevant tests or Daml Script files if they clarify intended behavior
- verify whether a guard already exists elsewhere in the workflow

Tests may confirm intent, but they do not override what production code allows.

### 8. Produce the report

Format the final output exactly as described in [references/report-formatting.md](references/report-formatting.md).

If `--file-output` is present, also write `daml-audit-report.md`.

## Non-Negotiable Rules

- Use Daml terms precisely: `signatory`, `observer`, `controller`, `stakeholder`, `maintainer`, `informee`, `disclosed contract`.
- Distinguish contract-model bugs from participant or topology misconfiguration.
- Treat Canton error codes as evidence about likely failure modes, not proof of a vulnerability by themselves.
- Do not assume `fetch`, `fetchByKey`, `lookupByKey`, and explicit disclosure have interchangeable authorization behavior.
- Do not assume caller-supplied timestamps are trustworthy when `getTime` is available.
- Do not assume a party who saw a contract through divulgence has durable knowledge of its future state.
- Do not promote "missing test" to a finding unless it hides a real security property gap.
- Findings must be concrete and exploitable within Daml and Canton semantics, not generic lint.

## What Good Findings Look Like

Good findings usually look like one of these:

- a party can unilaterally archive or rewrite another party's position because the stakeholder model is weak
- a supposedly one-time action is `nonconsuming` and can be replayed
- a flexible controller or delegation wrapper launders authority without checking the intended principal
- a mixed-party transaction divulges confidential contract data to an unintended witness
- `fetchByKey` or `lookupByKey` logic assumes visibility that the caller does not actually have
- duplicate signers or voters collapse a threshold or quorum model
- a choice uses caller-supplied time and lets someone fast-forward or backdate vesting, cancellation, timeout, or settlement
- a workflow can become permanently stuck because a state transition or recovery path is missing

## What Bad Findings Look Like

Reject or demote reports that rely on:

- "reentrancy", "gas griefing", or similar EVM-only language with no Daml analogue
- hypothetical attackers who cannot see or authorize the referenced contracts
- compile-time type errors presented as vulnerabilities
- privacy claims that ignore Daml's witness and divulgence rules
- vague "centralization" complaints without a violated invariant or trust boundary
