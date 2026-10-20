---
name: iac-guidelines
description: Theodo Cloud IaC Guild guidelines for Terraform and Terragrunt. Use when writing, modifying or reviewing .tf / .hcl files, designing layers or modules, or answering questions about IaC naming, versioning, structure, secrets and refactoring.
---

# Theodo Cloud IaC guidelines

The guideline documents live at the **root of this plugin** (`terraform/`, `terragrunt/`, `operations/`, `tooling/`), next to the `skills/` folder. They are the source of truth: **read the relevant file before applying or citing a rule**. The summary below is only a reminder.

Apply the rules in the context of the target project. Do not restructure an existing project just to enforce a style preference; follow its conventions where they are consistent, and mention deviations instead.

## Core rules

- **Layers**: a layer = one directory where you run `apply` = one state. It reflects a business need, has one owning team, and refreshes fast enough to contribute to.
- **Files**: `_settings.tf` for terraform/providers/backend, `output.tf` for the interface, reader-friendly names (`database.tf`). No `main.tf` (except the bootstrap layer).
- **Naming**: `snake_case`; no stuttering (`aws_route_table.public`, not `.public_route_table`); `this` / `these` in modules; plural names for collections, `for_each` loops and list outputs; every variable and output has a description.
- **Versions**: layers use exact versions (`= 1.6.0`) for Terraform and providers and commit `.terraform.lock.hcl`; modules use ranges (`~> 1.0`); remote modules are pinned to a version or tag.
- **Modules**: only to create a resource several times or to hide complexity; never a pass-through module; no `provider` block inside a module; no data sources in modules; README + terraform-docs.
- **Red flags**: `terraform workspace`, complex one-liners, `tfvars` instead of `locals`, shell scripts mixed with `.tf`, Kubernetes/Helm providers, monolayer repos.
- **Secrets**: never in plain text in code; reference existing secrets through data sources / the cloud secret manager.
- **Iteration**: `for_each` over `count`.
- **Cross-layer references**: data sources first, remote state second, hardcoded strings never.
- **Refactoring**: structural change only, generate `moved` blocks with `tfautomv`, apply an empty plan, then change configuration in a separate step.
- **Terragrunt (Context pattern)**: `root.hcl` + `module.hcl` + `inputs.hcl` + leaf `terragrunt.hcl`; one module per layer.

## Index: which file to read

| Topic | File |
| --- | --- |
| Red flags / what not to do (provider blocks in modules, data sources known at apply time...) | `terraform/donts.md` |
| Resource, variable, output, file naming | `terraform/terraform_naming.md` |
| Terraform / provider / module version constraints | `terraform/terraform_versioning.md` |
| Secrets in code and state | `terraform/handling_secrets.md` |
| `for_each` / `count` / `for` | `terraform/iterate_on_your_resources.md` |
| Optional object attributes | `terraform/optional_attributes.md` |
| Child module depth | `terraform/child_modules_depth_limit.md` |
| Pre-commit hooks | `terraform/pre-commits.md` |
| `.gitignore` | `terraform/gitignore.md` |
| Referencing other layers (Terraform) | `terraform/refering_to_resources_from_other_layers.md` |
| WYSIWYG pattern | `terraform/wysiwg_patterns.md` |
| Context pattern (Terragrunt) | `terragrunt/context_pattern.md` |
| Terragrunt decisions | `terragrunt/adr-terragrunt.md` |
| Referencing other layers (Terragrunt) | `terragrunt/refering_to_resources_from_other_layers.md` |
| Folder / layer structure | `operations/standard-best-iac-structure.md` |
| Refactoring and moves | `operations/refactoring.md` |
| tflint, tfautomv and other tools | `tooling/README.md`, `tooling/tflint.md`, `tooling/tfautomv.md` |

## When writing code

1. Identify the pattern in use (WYSIWYG or Context) and follow it.
2. Read the files from the index that match the change.
3. Write the code, then run `terraform fmt`, `terraform validate` and `tflint` (or `terragrunt hclfmt`) when available.
4. Never run `apply` or `destroy` unless explicitly asked; show the plan instead.
