# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Documentation only (Markdown, no build, lint or test commands): the Theodo Cloud IaC Guild's Terraform/Terragrunt guidelines. Content is advisory ("a proposition", not "absolute truth"), so keep a recommending tone.

## Layout

- `terraform/` – standards and red flags for vanilla Terraform (WYSIWYG pattern, naming, versioning, secrets, `donts.md`, `gitignore.md`, ...)
- `terragrunt/` – Terragrunt ADR and the Context pattern
- `operations/` – lifecycle topics (choosing an IaC structure, refactoring)
- `tooling/` – tflint, tfautomv and other tool notes
- `assets/` – files meant to be copied into consumer projects (e.g. `.pre-commit-config.yaml`)
- `.github/copilot-instructions.md` and `.github/prompts/` – condensed, actionable versions of the guidelines for Copilot (consumers copy these into their own repos)
- `skills/`, `agents/`, `.claude-plugin/`, `PLUGIN.md` – the `iac-guidelines` AI plugin (when present on your branch). The plugin *is* this repo: skills read the guideline files directly, nothing is duplicated.

## Keeping things in sync

A guideline lives in several places; when you add or change one, check each:

- `README.md` has a table of contents and per-section link lists (Patterns / Lifecycle / Standards / Red flags / Tooling). New documents must be linked there.
- `.github/copilot-instructions.md` summarizes the rules; update it when a rule changes. `.github/prompts/iac-audit.prompt.md` is independent of the `iac-audit` skill, so update both if the audit method changes.
- `skills/iac-guidelines/SKILL.md` has an index of guideline files: add a row for a new document.
- Keep the version aligned in `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`.
- Plugin skill frontmatter is limited to `name` + `description` (the subset Claude Code, Copilot CLI and Codex CLI all accept); tool-specific settings go in `agents/*.md` only.

## Conventions

- Branches are named like `docs/<issue>-<topic>`; commits follow Conventional Commits with a scope, e.g. `docs(terraform/donts): ...`. Changes land through PRs to `main`.
- Guideline vocabulary used across docs: *layer* (a Terraform state/root module), *WYSIWYG pattern* (vanilla Terraform), *Context pattern* (Terragrunt implementation). The "3 gold standards" and layer/module/naming standards in `README.md` are the core rules; do not contradict them in detail pages.
