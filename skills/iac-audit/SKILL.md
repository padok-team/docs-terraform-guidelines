---
name: iac-audit
description: Read-only audit of a Terraform / Terragrunt codebase against the Theodo Cloud IaC Guild guidelines, with a maturity level and risk-ranked findings. Use when asked to audit, assess or review the overall quality of an IaC repository or folder.
---

# IaC audit (read-only)

Audit the **path given by the user, otherwise the whole repository** against the Theodo Cloud IaC Guild standards. Never modify files.

The guideline documents are at the root of this plugin (`terraform/`, `terragrunt/`, `operations/`, `tooling/`); use the index in the `iac-guidelines` skill to pick the files relevant to each check and read them before judging. Subscription / tenant / AD group / resource IDs are public identifiers, not secrets. Only passwords, keys, certificates and tokens are sensitive.

## Evidence rules

- Every finding gives `file:line` **and** the guideline file it violates (e.g. `terraform/terraform_naming.md`).
- Give counts where possible ("7 of 12 variables lack a description"). Say "None found" rather than inventing findings.
- Unreadable or missing information is reported as unknown, not as a violation.

## What to check

1. **Structure & patterns**: WYSIWYG vs Context fits the team; layer design (business need, single owner, acceptable refresh time); folder and file naming; no monolayer repo.
2. **Naming**: snake_case, no stuttering, `this`/`these`, plural collections, no `main.tf`, descriptions on variables and outputs.
3. **Modules**: created for reuse or to hide complexity; no pass-through module; README / terraform-docs.
4. **Versioning**: exact versions in layers, ranges in modules, committed lock files, pinned remote modules.
5. **Quality & security**: pre-commit hooks OR CI running tflint / fmt / checkov (one of the two is enough); no plain-text secrets; no `provider` block in modules; no workspaces, tfvars sprawl, one-liners, scripts mixed with `.tf`.
6. **Cross-layer references**: data sources / remote state rather than hardcoded strings; documented dependencies.

## Maturity levels

5 Optimized · 4 Managed · 3 Defined · 2 Repeatable · 1 Initial. Assess each category (Architecture & Patterns, Standards & Quality, Security & Compliance, Automation & Tooling, Documentation & Process) and derive the overall level, weighting Security 35%, Architecture 25%, Standards 20%, Documentation 12%, Tooling 8%.

Risk levels: 🔴 Critical (security, outage, compliance) · 🟠 High · 🟡 Medium · 🟢 Low.

## Report format

1. **Overall**: maturity level, risk profile, scope, 2-3 sentence summary.
2. **Per category**: level, risk, findings (`file:line` + guideline file), gap to the next level, recommendations.
3. **Critical risks** (address immediately): impact and mitigation each.
4. **Roadmap**: ordered actions by risk reduction.
5. **Summary table**: category | maturity | risk | immediate actions, plus an overall row.
6. **Key takeaway**: one sentence.
