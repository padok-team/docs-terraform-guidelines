---
name: iac-upgrade-versions
description: Normalize and upgrade Terraform, Terragrunt and provider version constraints across a repository following the IaC Guild versioning rules, then check provider compatibility. Use when asked to upgrade providers, bump Terraform / Terragrunt versions or harmonize version pins.
---

# Upgrade and normalize version constraints

Read `terraform/terraform_versioning.md` first; it is the rule source. Scope: the path the user names, otherwise the repository. **Out of scope**: functional refactors, renames, behaviour changes.

## Inputs

Ask the user for the target versions (Terraform, Terragrunt, each provider), or propose the latest stable ones from the provider registry / releases and get confirmation before editing.

## Rules to apply

- **Layers** (anything you run `apply` in, including bootstrap): exact versions for Terraform and providers (`= x.y.z`), `.terraform.lock.hcl` committed. For Terragrunt, keep the pattern already used in `root.hcl`.
- **Modules**: ranges (`~>`); declare `required_providers` only for providers the module actually uses; never a `provider` configuration block.
- **Remote modules**: pinned to a version or tag.
- Keep existing provider sources. No workspaces or provisioners. Context pattern: one module per layer. Bootstrap stays a plain Terraform layer.

## Steps

1. **Inventory**: search `required_version`, `required_providers`, `terragrunt_version_constraint`, `terraform_version_constraint`, module `version` / `?ref=`; list where each constraint lives and which deviate from the rules.
2. **Update** layers, modules (`versions.tf` / `version.tf`; add the file if a module lacks it), and examples, with the agreed versions. Keep diffs minimal, no reformatting of unrelated code.
3. **Consistency sweep**: remove stale pins, check no provider block slipped into a module.
4. **Compatibility check** per provider/version pair: list the resource types used (`resource "<provider>_…"`), check each exists in that version's documentation, sample-check frequently used schemas for breaking changes (attribute types, renamed or deprecated attributes, required blocks, enum values). Apply trivial, unambiguous fixes inline; describe non-trivial migrations (with doc links) and leave that code unchanged.
5. **Validate**: `terraform fmt`, `terragrunt hclfmt`, `tflint`, `checkov` where configured; `terraform init -upgrade` + `terraform validate` in representative layers and modules; refresh and commit lock files for layers. Never run `apply`.

## Report

Constraints changed (before → after, per file), fixes applied, migrations left for humans, validation results, and README snippets that still mention old versions (update them if the user agrees). For structural follow-ups, prefer `tfautomv` moved blocks before changing configuration.
