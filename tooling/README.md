# Useful tooling for Terragrunt/Terraform

The goal of this page is to list some useful tooling for Terragrunt/Terraform.

- [Useful tooling for Terragrunt/Terraform](#useful-tooling-for-terragruntterraform)
  - [Code quality](#code-quality)
  - [Code security](#code-security)
  - [Documentation](#documentation)
  - [Readability of plan and apply](#readability-of-plan-and-apply)
  - [How to operate this tools](#how-to-operate-this-tools)

## Code quality

> Good code quality is a must-have for any project

- [tflint](tflint.md) ✨ - Linter for Terraform
  - [Default configuration](tflint.md#default-configuration)
  - We recommend this one because it can check a wide range of cloud providers as well as Terraform code
- [terraform fmt](https://developer.hashicorp.com/terraform/cli/commands/fmt) - Rewrites all Terraform configuration files to a canonical format
  - Example usage : `terraform fmt -recursive -diff -write=true`
- [terragrunt hclfmt](https://terragrunt.gruntwork.io/docs/reference/cli-options/#hclfmt) - Rewrites all Terragrunt configuration files to a canonical format
  - Example usage : `terragrunt hclfmt`

## Code security

> Left shift security related tasks as much as possible

- [checkov](https://github.com/bridgecrewio/checkov) ✨ - Static code analysis tool for infrastructure-as-code
  - Example : `checkov -d . --framework terraform --skip-file baseline.skip`
  - We recommend this one because it can check a wide range of cloud providers as well as Terraform code
- [tfsec](https://github.com/aquasecurity/tfsec) - Static analysis powered security scanner for your terraform code
  - Example : `tfsec .`
- [terrascan](https://runterrascan.io/) - Detect compliance and security violations across Infrastructure as Code to mitigate risk before provisioning cloud native infrastructure
  - Example : `terrascan scan -i terraform -d .`

## Documentation

> Documentation is a must-have for any project

- [terraform-docs](https://github.com/terraform-docs/terraform-docs) - Generate documentation from Terraform modules in various output formats
  - Example : `terraform-docs markdown .`

## Readability of plan and apply

> When working with Terraform and even more so for Terragrunt, reading plan can be a pain.
> Terraform is not fixing it any time soon : [Github issue on concise plan](https://github.com/hashicorp/terraform/issues/10507)

- grep ✨
  - `terraform plan -no-color | grep -E '(^.*[#~+-] .*|^[[:punct:]]|Plan|Changes)'`
  - We recommend this one because it's simple and efficient
- [tfnotify](https://github.com/mercari/tfnotify)
- [tftools](https://github.com/containerscrew/tftools)
- [tf-summarize](https://github.com/dineshba/tf-summarize)

## How to operate this tools

- [pre-commit](https://pre-commit.com/) - A framework for managing and maintaining multi-language pre-commit hooks

For terraform fmt, terragrunt hcl and checkov you can use the following configuration :

```yaml
repos:
  - repo: https://github.com/antonbabenko/pre-commit-terraform
    rev: v1.77.0
    hooks:
      - id: terraform_fmt
      - id: terragrunt_fmt
      - id: terraform_checkov
        args:
          - --args=--quiet    
          - --args=--framework=terraform
      - id: terraform_providers_lock
        args:
          - --hook-config=--mode=only-check-is-current-lockfile-cross-platform
```

For tflint check [here](tflint.md#how-to-use-it)

- [CI/CD](tbd) - Run these tools in your CI/CD pipeline
