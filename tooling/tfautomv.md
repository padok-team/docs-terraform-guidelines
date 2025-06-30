# tfautomv - Terraform Refactoring Automation

`tfautomv` (Terraform auto-move) is a refactoring helper that automatically generates `moved` blocks and `terraform state mv` commands when you restructure your Terraform code. It helps make structural changes to your Terraform codebase without losing track of resource state.

- [tfautomv - Terraform Refactoring Automation](#tfautomv---terraform-refactoring-automation)
  - [What is tfautomv?](#what-is-tfautomv)
  - [Installation](#installation)
    - [Recommended: Homebrew (macOS/Linux)](#recommended-homebrew-macoslinux)
    - [Shell Script (macOS/Linux)](#shell-script-macoslinux)
    - [Manual Download](#manual-download)
    - [asdf Version Manager](#asdf-version-manager)
  - [Core Concepts](#core-concepts)
  - [Basic Usage](#basic-usage)
    - [Simple Refactoring](#simple-refactoring)
    - [Target Specific Directory](#target-specific-directory)
    - [Generate Commands Instead of Blocks](#generate-commands-instead-of-blocks)
    - [Execute Commands Directly](#execute-commands-directly)
  - [Advanced Usage](#advanced-usage)
    - [Multiple Directories](#multiple-directories)
    - [Custom Output Formats](#custom-output-formats)
    - [Performance Optimization](#performance-optimization)
    - [Using with Pre-generated Plans](#using-with-pre-generated-plans)
  - [Tool Integration](#tool-integration)
    - [Terragrunt Support](#terragrunt-support)
    - [OpenTofu Support](#opentofu-support)
    - [Pre-commit Integration](#pre-commit-integration)
  - [Ignoring Differences](#ignoring-differences)
    - [When to Use --ignore](#when-to-use---ignore)
    - [Available Ignore Rules](#available-ignore-rules)
      - [Everything Rule](#everything-rule)
      - [Whitespace Rule](#whitespace-rule)
      - [Prefix Rule](#prefix-rule)
      - [Nested Attributes](#nested-attributes)
    - [Best Practices for Ignoring](#best-practices-for-ignoring)
  - [Best Practices](#best-practices)
    - [Recommended Workflow](#recommended-workflow)
    - [Good Use Cases](#good-use-cases)
    - [Cases to Avoid](#cases-to-avoid)
  - [Tips and Tricks](#tips-and-tricks)
    - [Debugging and Verbosity](#debugging-and-verbosity)
    - [CI/CD Integration](#cicd-integration)
    - [Enterprise Environments](#enterprise-environments)
  - [Common Issues and Solutions](#common-issues-and-solutions)

## What is tfautomv?

When you move or rename resources in your Terraform code, Terraform loses track of the resource's state. The next time you run `terraform plan`, it will plan to delete the old resource and create a "new" one - even though they're the same resource with a different name or location.

`tfautomv` solves this by:

1. Running `terraform plan` to see what Terraform wants to create and delete
2. Comparing create/delete pairs to find resources that are actually the same
3. Generating `moved` blocks or `terraform state mv` commands to preserve state

## Installation

### Recommended: Homebrew (macOS/Linux)

```bash
brew install busser/tap/tfautomv
```

### Shell Script (macOS/Linux)

> **Security Note:** Always review remote installation scripts before executing them to ensure they are safe.

```bash
curl -sSfL https://raw.githubusercontent.com/busser/tfautomv/main/install.sh -o install.sh
less install.sh   # Review the script contents
sh install.sh
```

### Manual Download

Download the appropriate binary from the [GitHub releases page](https://github.com/busser/tfautomv/releases) and place it in your PATH.

### asdf Version Manager

```bash
asdf plugin add tfautomv https://github.com/busser/asdf-tfautomv.git
asdf install tfautomv latest
asdf global tfautomv latest
```

## Core Concepts

- **Pure Refactoring**: tfautomv is designed for structural changes without modifying actual infrastructure
- **State Preservation**: Keeps Terraform state intact when moving resources
- **Automatic Detection**: Intelligently matches create/delete pairs that represent the same resource
- **Multiple Output Formats**: Can generate `moved` blocks or shell commands

## Basic Usage

### Simple Refactoring

Run in any directory where you would normally run `terraform plan`:

```bash
tfautomv
```

This will:

1. Run `terraform init`, `terraform refresh`, and `terraform plan`
2. Generate `moved` blocks in a `moves.tf` file
3. Show a summary of what it found

### Target Specific Directory

```bash
tfautomv ./production
```

### Generate Commands Instead of Blocks

```bash
tfautomv --output=commands
# or short form
tfautomv -o commands
```

### Execute Commands Directly

```bash
tfautomv -o commands | sh
```

## Advanced Usage

### Multiple Directories

Move resources across different Terraform modules:

```bash
tfautomv ./production/main ./production/backup -o commands
```

**Note**: Cross-module moves only work with `--output=commands`, not with `moved` blocks.

### Custom Output Formats

- `--output=blocks` (default): Generate `moved` blocks
- `--output=commands`: Generate `terraform state mv` commands

### Performance Optimization

Skip expensive operations when iterating:

```bash
# Skip initialization
tfautomv --skip-init

# Skip refresh
tfautomv --skip-refresh

# Skip both (short form)
tfautomv -sS
```

### Using with Pre-generated Plans

Perfect for CI/CD or when you already have plan files:

```bash
# Generate plan first
terraform plan -out=tfplan.bin

# Use existing plan
tfautomv --preplanned

# Custom plan file name
terraform plan -out=my-plan.bin
tfautomv --preplanned --preplanned-file=my-plan.bin

# JSON plans work too
terraform show -json tfplan.bin > tfplan.json
tfautomv --preplanned --preplanned-file=tfplan.json
```

## Tool Integration

### Terragrunt Support

```bash
tfautomv --terraform-bin=terragrunt
```

### OpenTofu Support

```bash
tfautomv --terraform-bin=tofu
```

### Pre-commit Integration

Add to your `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/busser/tfautomv
    rev: v0.7.0 # Use the latest version
    hooks:
      - id: tfautomv
        args: ["--output=blocks"]
```

## Ignoring Differences

### When to Use --ignore

Use `--ignore` when providers transform attribute values in ways that prevent tfautomv from matching resources:

- Provider normalizes JSON/XML formatting
- Provider adds computed fields not in configuration
- Provider transforms string casing or formatting

### Available Ignore Rules

#### Everything Rule

Ignore any difference in an attribute:

```bash
tfautomv --ignore="everything:aws_s3_bucket:arn"
```

#### Whitespace Rule

Ignore whitespace differences (useful for JSON/XML):

```bash
tfautomv --ignore="whitespace:azurerm_api_management_policy:xml_content"
```

#### Prefix Rule

Ignore specific prefixes:

```bash
tfautomv --ignore="prefix:google_storage_bucket_iam_member:bucket:b/"
```

#### Nested Attributes

Reference nested attributes with dots:

```bash
tfautomv --ignore="everything:aws_instance:tags.Environment"
```

### Best Practices for Ignoring

**✅ Good Examples:**

```bash
# Provider normalizes JSON formatting
tfautomv --ignore="whitespace:aws_iam_policy:policy"

# Provider adds computed ARN field
tfautomv --ignore="everything:aws_s3_bucket:arn"

# Provider normalizes bucket path
tfautomv --ignore="prefix:google_storage_bucket:name:projects/my-project/buckets/"
```

**❌ Avoid These:**

```bash
# DON'T ignore intentional changes
tfautomv --ignore="everything:aws_instance:tags"  # You changed tags!

# DON'T mix refactoring with infrastructure changes
```

## Best Practices

### Recommended Workflow

1. **Make structural changes only**: Rename resources, move between modules
2. **Run tfautomv**: Generate moves without any configuration changes
3. **Apply moves**: Should result in empty plan (no infrastructure changes)
4. **Make configuration changes**: In separate operation, modify attributes

### Good Use Cases

- ✅ Renaming resources: `aws_instance.web` → `aws_instance.web_server`
- ✅ Moving between modules: `aws_instance.web` → `module.ec2.aws_instance.web`
- ✅ Converting to `for_each` loops
- ✅ Module restructuring

### Cases to Avoid

- ❌ Changing resource configuration while renaming
- ❌ Adding/removing attributes during moves
- ❌ Provider-specific transformations that change infrastructure

## Tips and Tricks

### Debugging and Verbosity

Increase verbosity to understand what tfautomv is doing:

```bash
# Show attribute paths and comparison details
tfautomv -vvv

# See exactly which attributes differ
tfautomv --verbosity=3
```

### CI/CD Integration

Use pre-generated plans in CI/CD pipelines:

```bash
# In CI pipeline
terraform plan -out=tfplan.bin -no-color > plan.txt
terraform show -json tfplan.bin > tfplan.json

# Later stage or local
tfautomv --preplanned --preplanned-file=tfplan.json
```

### Enterprise Environments

For complex enterprise setups with remote state:

```bash
# Use environment variables for Terraform configuration
TF_CLI_ARGS_plan="-var-file=production.tfvars" tfautomv

# Skip expensive operations in large environments
tfautomv -sS --preplanned
```

## Common Issues and Solutions

**Issue**: "No moves found" despite obvious renames

- **Solution**: Check for attribute differences with `-vvv`, use `--ignore` if needed

**Issue**: tfautomv suggests moves that would change infrastructure

- **Solution**: Don't apply those moves; separate refactoring from configuration changes

**Issue**: Cross-module moves not working

- **Solution**: Use `--output=commands` instead of `moved` blocks

**Issue**: Slow performance with large state

- **Solution**: Use `--skip-init --skip-refresh` or `--preplanned`

**Issue**: Provider transforms attributes

- **Solution**: Use appropriate `--ignore` rules for provider-specific transformations

For more detailed information and examples, visit the [official tfautomv repository](https://github.com/busser/tfautomv).
