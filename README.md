# Canton Guard by SCAS

A security agent for finding potential vulnerabilities in Daml.

Built for:

- Daml developers who want a security check before shipping workflow changes
- Canton teams reviewing authorization, privacy, and contract-key logic
- Security researchers looking for fast, Daml-native audit hypotheses before manual review

## Installation
Open claude/codex and paste this prompt
```Install this skill: https://github.com/SCAuditStudio/CantonGuard```

To use the skill in github copilot, clone the repo to the `.github/copilot-skills` folder.

## Usage

    Use $canton-guard-by-scas to review the codebase

    Use $canton-guard-by-scas on specified .daml files

    Use $canton-guard-by-scas --include-tests when workflow intent lives in Daml Script or test modules

    Use ./github/copilot-skills/canton-guard-by-scas to review the codebase

## Tips

- Target hot modules. Point the skill at the contracts or workflows you are actively changing instead of scanning a large repo blindly.
- Run more than once. Different passes often surface different authorization and privacy issues.
- Include tests when needed. In Daml, test and script modules often reveal the intended party flow, deadline logic, and failure cases.

## Contact
If you are looking to explore strategies for securing your project, reach out for a chat on [scauditstudio.com](https://scauditstudio.com/contact).
