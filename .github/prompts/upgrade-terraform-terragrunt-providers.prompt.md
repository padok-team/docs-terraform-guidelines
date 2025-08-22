---
mode: agent
description: "Guided prompt to standardize Terraform/Terragrunt constraints and provider pins across this repo following Theodo Cloud IaC Guild best practices and padok azure-library conventions."
tools:
  [
    "codebase",
    "semantic_search",
    "file_search",
    "grep_search",
    "read_file",
    "list_dir",
  ]
---

## Goal

Upgrade and normalize version constraints everywhere in this repository:

- Terraform CLI: "~> 1.10"
- Terragrunt: "~> 0.77" (starters root.hcl)
- Providers when used:
  - azurerm: "~> 4"
  - azapi: "~> 2" (where referenced)
  - random: "~> 3" (where referenced)
  - azuread: "~> 3" (where referenced)

Keep modules flexible and deterministic enough per repo conventions. Preserve structure (Context pattern with Terragrunt for starters; bootstrap exception). Do not introduce workspaces or provisioners.

## Scope and targets

Update constraints in these locations:

- Starters
  - starter/\*/layers/root.hcl → terraform_version_constraint and terragrunt_version_constraint
  - starter/\*/layers/bootstrap/\_settings.tf → terraform required_version and required_providers
- Modules
  - modules/\*/(versions.tf|version.tf) → terraform required_version and required_providers
  - If a module lacks a versions file, add one with the correct pins for the providers actually used by that module
- Examples
  - modules/\*/examples/\*\*/(\_settings.tf|main.tf) → terraform required_version and required_providers where applicable

Out of scope: functional refactors, resource renames, or behavioral changes.

## Guardrails (IaC Guild standards)

- Use snake_case names; avoid stuttering in resource names
- Do not introduce terraform workspaces or shell provisioners
- Modules may declare required_providers in the terraform block but must not contain provider configuration blocks
- Keep one module per layer in Terragrunt (Context pattern)
- Bootstrap remains a plain Terraform layer until remote state exists
- Prefer ~> constraints for modules; commit .terraform.lock.hcl in layers

## Step-by-step

### 1) Inventory current constraints

- Search for required_version, required_providers, and terragrunt_version_constraint across the repo
- Note occurrences of "~> 1", "~> 1.0", "~> 1.10.1", or older Terragrunt ranges

### 2) Starters updates

- In each starter’s layers/root.hcl:
  - terraform_version_constraint = "~> 1.10"
  - terragrunt_version_constraint = "~> 0.77"
- In each starter’s bootstrap/\_settings.tf:
  - terraform.required_version = "~> 1.10"
  - required_providers.azurerm.version = "~> 4"

### 3) Modules updates

- In each module versions.tf (or version.tf):
  - terraform.required_version = "~> 1.10"
  - required_providers pins only for providers actually used in that module:
    - azurerm "~> 4"; azapi "~> 2"; random "~> 3"; azuread "~> 3"
- If a module lacks a versions file, add it with these constraints

### 4) Examples updates

- Normalize example \_settings.tf or main.tf files to terraform.required_version = "~> 1.10"
- If examples declare providers locally, pin them consistently as above

### 5) Consistency sweep

- Remove lingering "~> 1" or "~> 1.10.1" pins
- Ensure no provider blocks are added inside modules beyond required_providers metadata
- Keep existing provider sources (hashicorp/azurerm, azure/azapi, hashicorp/random, hashicorp/azuread)

### 6) Validation and hygiene

- Run: terraform fmt, terragrunt hclfmt, tflint, checkov (where configured)
- Run: terraform init -upgrade and terraform validate in representative layers (bootstrap, a couple of modules, and one example)

### 7) Provider compatibility checks (generic)

- Build a list of providers and versions in scope from required_providers across changed files (and any Terragrunt-generated provider files).
- For each provider-version pair:
  - Enumerate distinct resource types used in the repo for that provider (regex: `resource "<provider>_[a-z0-9_]+"`).
  - Cross-check each resource type exists in that provider/version’s documentation; flag unknown/renamed resources.
  - Sample-check schemas for frequently used resources/blocks to catch breaking changes:
    - Attribute type drift (e.g., quoted numerics/booleans vs native types)
    - Renamed/deprecated attributes or required blocks
    - Enum value changes (e.g., SKU or mode names)
  - Apply trivial, low-risk fixes inline (e.g., change "100" -> 100; update renamed attribute names where unambiguous).
  - Note any non-trivial migrations with a short rationale and links to docs/issues (leave code unchanged unless clearly safe).
- Run terraform init -upgrade and terraform validate in representative layers/modules to confirm compatibility.
- Document any code changes made as part of the sweep.

## Acceptance criteria

- [ ] All starters have "~> 1.10" (Terraform) and "~> 0.77" (Terragrunt)
- [ ] All modules declare Terraform "~> 1.10" in versions.tf (or version.tf)
- [ ] Providers pinned per module usage: azurerm "~> 4"; azapi "~> 2"; random "~> 3"; azuread "~> 3"
- [ ] Examples normalized to Terraform "~> 1.10"
- [ ] No stray "~> 1", "~> 1.0", or "~> 1.10.1" remain
- [ ] terraform fmt and terragrunt hclfmt are clean; validate passes in sampled layers
- [ ] Provider compatibility sweep completed for all pinned providers/versions; trivial fixes applied; any non-trivial migrations documented with references

## Notes and tips

- Keep changes minimal and deterministic; avoid mass reformatting unrelated code
- Respect existing layer/module structure and naming conventions
- If a provider is unused in a module, do not declare it in required_providers
- For future refactors, prefer tfautomv for moved blocks before changing configuration

## Optional follow-ups (docs)

- Update README snippets that reference older versions (e.g., Terragrunt "~> 0.73" or Terraform "~> 1.10.1") to reflect the new standards
