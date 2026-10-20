# `iac-guidelines` plugin

Packages these guidelines for AI coding tools. The plugin **is this repository**: the skills read the guideline documents (`terraform/`, `terragrunt/`, `operations/`, `tooling/`) directly, so there is nothing to keep in sync.

## What it provides

| Component | Name | Purpose |
| --- | --- | --- |
| Skill | `iac-guidelines` | Core rules + index of the guideline files; used when writing or reviewing `.tf` / `.hcl` |
| Skill | `iac-audit` | Read-only maturity and risk audit, findings cite `file:line` + guideline file |
| Skill | `iac-c4-diagrams` | C4 Context + Container Mermaid diagrams from a Terragrunt folder |
| Skill | `iac-architecture-diagram` | Mermaid architecture flowchart (layers, CIDRs, dependencies) + write-up |
| Skill | `iac-upgrade-versions` | Normalize / upgrade Terraform, Terragrunt and provider constraints, with compatibility checks |
| Agent | `iac-developer` | Writes / modifies IaC with the guidelines preloaded |
| Agent | `iac-auditor` | Read-only audit agent |

## Compatibility

One source for all tools: `.claude-plugin/` (manifest + marketplace), `skills/` and `agents/`. Copilot CLI and Codex CLI read the Claude Code layout as a fallback, so nothing is duplicated.

| Tool | Skills | Agents |
| --- | --- | --- |
| Claude Code | yes | yes (`agents/`) |
| GitHub Copilot CLI | yes | yes, same `agents/*.md` (named `iac-guidelines:iac-auditor`; `Read, Grep, Glob` map to `view, grep, glob`) |
| Codex CLI | yes | **no**: Codex plugins have no agents. Use the `iac-guidelines` skill for coding and `iac-audit` for audits |

IDEs that do not load plugins (e.g. the Codex IDE extension, VS Code Copilot, which reads skills from `.github/skills/` or `.claude/skills/`): copy the `skills/` folders into your project's skill directory, together with the `terraform/`, `terragrunt/`, `operations/` and `tooling/` folders they reference.

## Install

```bash
# Claude Code
claude plugin marketplace add padok-team/docs-terraform-guidelines
claude plugin install iac-guidelines@theodo-iac

# GitHub Copilot CLI
copilot plugin install padok-team/docs-terraform-guidelines

# Codex CLI
codex plugin marketplace add padok-team/docs-terraform-guidelines
codex plugin add iac-guidelines@theodo-iac
```

Local checkout: replace the repository with `./` (Claude Code: `claude --plugin-dir .`).

## Use

- Claude Code: `/iac-guidelines:iac-audit path/to/layer`, or ask for the `iac-auditor` / `iac-developer` agent.
- Copilot CLI: `/iac-guidelines:iac-audit`, or pick the `iac-auditor` / `iac-developer` agent.
- Codex: ask for a skill by name (`iac-audit`, `iac-guidelines`, `iac-c4-diagrams`...).

## Maintenance

- Skill frontmatter is limited to `name` + `description`, the subset all three tools accept. Tool-specific settings (`tools`, `skills`) go in the agent files only.
- When adding a guideline document, add a row to the index in `skills/iac-guidelines/SKILL.md`.
- Keep the version aligned in `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`.