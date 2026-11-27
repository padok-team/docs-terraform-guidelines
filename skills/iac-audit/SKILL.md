---
name: iac-audit
description: Read-only audit of a Terraform / Terragrunt codebase against the Theodo Cloud IaC Guild guidelines, with a maturity level and risk-ranked findings. Use when asked to audit, assess or review the overall quality of an IaC repository or folder.
---

# IaC audit (read-only)

Audit the **path given by the user, otherwise the whole repository** against the Theodo Cloud IaC Guild standards. Never modify files. The only commands allowed are `command -v guacamole` and `guacamole static ...` (see below).

The guideline documents are at the root of this plugin (`terraform/`, `terragrunt/`, `operations/`, `tooling/`); use the index in the `iac-guidelines` skill to pick the files relevant to each check and read them before judging. Subscription / tenant / AD group / resource IDs are public identifiers, not secrets. Only passwords, keys, certificates and tokens are sensitive.

## Guacamole

[Guacamole](https://github.com/padok-team/guacamole) statically checks Terraform / Terragrunt code (HCL parsing only). It is the evidence base for the Naming, Modules and Versioning points it covers; do not re-audit those by hand. Keep manual review for the rest (structure, layer design, security, cross-layer references, CI, docs, lock files).

- **Run order**: if `command -v guacamole` succeeds, run `guacamole static -v -p <audited path>` first. Otherwise use the Guacamole output the user provided. Otherwise audit everything manually and state "Guacamole not available" in the report.
- **Allowed commands**: only `command -v guacamole` and `guacamole static ...`. Never `state`, `profile` or `ci` modes, never `terraform` / `terragrunt` or any other command.
- **Output**: per check `✅|❌ <CHECK_ID> - <name>`, with `-v` each error as `path:line`, final `Score: x% (ok/total)`.
- **Layers**: Guacamole checks every folder with `.tf`, layers included. Discard `TF_MOD_003`, `TF_NAM_001`, `TF_NAM_005` hits on layers (layers use exact versions; `this`/`these` rules target modules). Report how many were discarded and why.
- **Waivers**: list every `# guacamole-ignore:<CHECK_ID> <reason>` comment and `.guacamoleignore` entry (`file:line` + reason). A waiver without a reason is a 🟡 finding.
- **Score**: shown for information only; it does not change the maturity level.

| Check | Guideline file |
|---|---|
| `TF_MOD_001` remote module pinned, `TF_MOD_003` `~>` provider constraints in modules | `terraform/terraform_versioning.md` |
| `TF_MOD_002` no provider in module, `TF_VAR_001` variable description, `TF_VAR_002` no `any` type, `TF_DAT_001` data source depending on apply-time value | `terraform/donts.md` |
| `TF_NAM_001` to `TF_NAM_005` this/these, snake_case, stuttering, plural variables | `terraform/terraform_naming.md` |
| `TG_DRY_001`, `TG_DRY_002`, `TG_ARC_001`, `TG_INP_001` duplicate inputs, useless locals, leaf layer, inputs match module variables | `terragrunt/context_pattern.md` |

## Evidence rules

- Every finding gives `file:line` **and** the guideline file it violates (e.g. `terraform/terraform_naming.md`). Guacamole findings: `file:line · <CHECK_ID> · <guideline file>`.
- Give counts where possible ("7 of 12 variables lack a description"). Say "None found" rather than inventing findings.
- Unreadable or missing information is reported as unknown, not as a violation.

## What to check

1. **Structure & patterns**: WYSIWYG vs Context fits the team; layer design (business need, single owner, acceptable refresh time); folder and file naming; no monolayer repo.
2. **Naming**: snake_case, no stuttering, `this`/`these`, plural collections, no `main.tf`, descriptions on variables and outputs. Guacamole-covered points come from Guacamole when available.
3. **Modules**: created for reuse or to hide complexity; no pass-through module; README / terraform-docs. Guacamole-covered points (pinned remote modules, no provider block) come from Guacamole when available.
4. **Versioning**: exact versions in layers, ranges in modules, committed lock files, pinned remote modules. Guacamole-covered points come from Guacamole when available.
5. **Quality & security**: pre-commit hooks OR CI running tflint / fmt / checkov (one of the two is enough); Guacamole in the project's CI (guacamole-action, GitLab job) is a positive signal in Automation & Tooling, not required; no plain-text secrets; no `provider` block in modules; no workspaces, tfvars sprawl, one-liners, scripts mixed with `.tf`.
6. **Cross-layer references**: data sources / remote state rather than hardcoded strings; documented dependencies.

## Maturity levels

5 Optimized · 4 Managed · 3 Defined · 2 Repeatable · 1 Initial. Assess each category (Architecture & Patterns, Standards & Quality, Security & Compliance, Automation & Tooling, Documentation & Process) and derive the overall level, weighting Security 35%, Architecture 25%, Standards 20%, Documentation 12%, Tooling 8%.

Risk levels: 🔴 Critical (security, outage, compliance) · 🟠 High · 🟡 Medium · 🟢 Low.

## Report format

1. **Overall**: maturity level, risk profile, scope, 2-3 sentence summary, Guacamole score (information only) or "Guacamole not available".
2. **Per category**: level, risk, findings (`file:line` + guideline file), layer false positives discarded (count), waivers list, gap to the next level, recommendations.
3. **Critical risks** (address immediately): impact and mitigation each.
4. **Roadmap**: ordered actions by risk reduction.
5. **Summary table**: category | maturity | risk | immediate actions, plus an overall row.
6. **Key takeaway**: one sentence.
