# Tflint

The goal of this page is to list some useful information about [tflint](https://github.com/terraform-linters/tflint)

## Default configuration

Within the `.tflint.hcl` file, you can define a default configuration for all your projects.

```hcl
plugin "terraform" {
  enabled = true
  source  = "github.com/terraform-linters/tflint-ruleset-terraform"
  preset  = "all"
}

rule "terraform_naming_convention" {
  enabled = true
}

# Change it depending on your cloud providers
plugin "aws" {
  enabled = true
  source  = "github.com/terraform-linters/tflint-ruleset-aws"
}
```

- [AWS](https://github.com/terraform-linters/tflint-ruleset-aws)
- [Azure](https://github.com/terraform-linters/tflint-ruleset-azurerm)
- [GCP](https://github.com/terraform-linters/tflint-ruleset-google)

## How to use it

- In the console : `tflint --recursive -f compact`
- In terragrunt 
  - Create an after_hook script for the validate command (Example below 👇)
  - Run `terragrunt run-all validate`

```hcl
terraform {
  after_hook "validate_tflint" {
    commands = ["validate"]
    execute = [
      "sh", "-c", <<EOT
        echo "Run tflint for layer '${path_relative_to_include()}'..."
        (tflint --config="${get_repo_root()}/.tflint.hcl" --force --color -f compact)
        error_code=$?
        echo "Run tflint for layer '${path_relative_to_include()}'...DONE\n"
        exit $error_code
      EOT
    ]
  }
}
```
