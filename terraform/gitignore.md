# Gitignore recommendations for Terraform

## TL;DR

- Ignore local Terraform files, state, saved plans, and files that may contain secrets.
- Commit `.terraform.lock.hcl` files in layers.
- Only commit variable values and overrides when they are intentionally shared and contain no secrets.

## Installation

Copy the [`.gitignore` template](../assets/.gitignore) to the root of your Terraform repository. If you already have a `.gitignore`, merge these rules into it.

The template is based on the [Toptal Terraform template](https://www.toptal.com/developers/gitignore/api/terraform), with additional rules for saved plans. Its rules apply to files in nested layers as well as at the repository root.

## Files to ignore

| Patterns | Reason |
| --- | --- |
| `.terraform/` | Terraform downloads providers and modules and stores local working data here. Recreate these directories with `terraform init`. |
| `*.tfstate`, `*.tfstate.*` | State and its backups can contain secrets. Use a protected remote backend instead of Git. |
| `crash.log`, `crash.*.log` | Crash logs are local debugging artifacts and may contain sensitive information. |
| `*.tfvars`, `*.tfvars.json` | Variable values can contain secrets or local settings, including automatically loaded `.auto.tfvars` files. |
| `override.tf`, `override.tf.json`, `*_override.tf`, `*_override.tf.json` | Overrides are usually local changes to resource configuration. |
| `tfplan`, `*.tfplan`, `*.tfplan.json` | Saved plans and JSON exports can contain sensitive values, even when Terraform hides them in terminal output. |
| `.terraformrc`, `terraform.rc` | Local CLI configuration can contain credentials and machine-specific settings. |

Terraform lets you choose any filename for a saved plan. Use a name covered by the template, such as `terraform plan -out=production.tfplan`, or add a rule matching your chosen filename. Apply the same rule to exported plans, for example `terraform show -json production.tfplan > production.tfplan.json`.

## Files to commit

Commit your Terraform configuration (`.tf` and `.tf.json`) and the `.terraform.lock.hcl` files in your layers. The lock file records provider selections and checksums so that collaborators and CI use consistent dependencies. See [Terraform versioning](terraform_versioning.md#terraform-lock-files-in-layers).

Some projects intentionally share non-sensitive variable values or overrides. Review their contents first, then add an explicit exception **after** the exclusion rules:

```gitignore
# Shared variable values without secrets
!environments/development.tfvars

# Intentionally shared override without secrets
!example_override.tf
```

These exceptions allow Git to track the named files; they do not check for secrets. Keep secrets in a secret management tool and follow the [handling secrets guideline](handling_secrets.md).

## Already tracked files

A `.gitignore` only affects untracked files. Adding a rule does not remove a file from Git or from its history.

For a file that should no longer be tracked, remove it from the index while keeping your local copy:

```bash
git rm --cached path/to/file
```

Commit that removal together with the relevant ignore rule. If credentials were committed, rotate them and address their exposure in Git history; adding an ignore rule does not resolve the leak.
