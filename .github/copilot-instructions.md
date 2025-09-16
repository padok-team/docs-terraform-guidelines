# Infrastructure as Code (IaC) Development Instructions

This workspace follows Theodo Cloud IaC Guild best practices for Terraform and Terragrunt development. These instructions guide GitHub Copilot to generate code that aligns with our standards and recommendations.

## Core Principles

### WYSIWYG vs Context Pattern Decision Making

- **WYSIWYG Pattern**: Use for smaller teams (1-2 people), synchronous collaboration, and when you need ease of exploration where "what you see is what you get"
- **Context Pattern**: Use for medium to large teams, asynchronous collaboration, when you have code duplication issues between layers calling the same modules
- **Key Trade-off**: WYSIWYG provides simplicity and ease of exploration but leads to code duplication; Context pattern (Terragrunt) reduces duplication but adds complexity

### The 3 Gold Standards for Infrastructure Codebase

1. Your layer design reflects a business need
2. Only one team contributes/has ownership of a layer (clear vision of contribution & ownership)
3. Your state refresh time is acceptable for a contribution purpose

## Project Structure & Patterns

### WYSIWYG Pattern (Terraform)

```
.
├── environment/
│   ├── dev/
│   │   ├── _settings.tf
│   │   ├── output.tf
│   │   ├── database.tf      # Calls local database module
│   │   └── environment.tf   # Calls local environment module
│   ├── preproduction/
│   └── production/
├── application/
│   ├── app_1/
│   │   ├── dev/
│   │   ├── preproduction/
│   │   └── production/
│   ├── app_2/
│   └── app_3/
└── modules/
    ├── environment/
    └── database/
```

### Context Pattern (Terragrunt)

```
.
├── environment
│   ├── module.hcl
│   ├── dev
│   │   ├── inputs.hcl
│   │   └── terragrunt.hcl
│   ├── preproduction
│   │   ├── inputs.hcl
│   │   └── terragrunt.hcl
│   └── production
│       ├── inputs.hcl
│       └── terragrunt.hcl
├── application
│   ├── app_1
│   │   ├── module.hcl
│   │   ├── dev
│   │   │   ├── inputs.hcl
│   │   │   └── terragrunt.hcl
│   │   ├── preproduction
│   │   │   ├── inputs.hcl
│   │   │   └── terragrunt.hcl
│   │   └── production
│   │       ├── inputs.hcl
│   │       └── terragrunt.hcl
│   ├── app_2
│   └── app_3
├── common.hcl
└── _settings.hcl
```

#### Context Pattern Benefits

- **DRY Principle**: Shared configuration eliminates duplication across layers
- **Environment Consistency**: Same module configuration across environments
- **Scalable**: Easy to add new environments or components
- **Input Heaven**: Define inputs throughout the tree structure
- **Uniqueness**: Each layer defines one and only one module
- **Ease of Refactoring**: Easy to split layers to match project needs

#### Context Pattern Structure Rules

- `common.hcl`: Store default values for every layer (e.g., tenant ID, database size)
- `_settings.hcl`: Define Terraform version, providers, and backend configuration for all layers
- `module.hcl`: Define the module used by subsequent layers
- `inputs.hcl`: Define layer-specific inputs only
- `terragrunt.hcl`: Define Terragrunt configuration with includes
- Each layer defines one and only one module to enforce clear separation

#### Bootstrap Layer Exception

The **bootstrap layer** is an intentional exception to the Context Pattern structure:

- **Purpose**: Creates the initial storage account/bucket for Terraform state before Terragrunt can be used
- **Structure**: Uses traditional Terraform structure (not Terragrunt) since it must run before the backend is available
- **Naming**: Can use `main.tf` or descriptive names like `bootstrap.tf`
- **State**: Typically uses local state initially, then migrated to remote backend once created
- **Environment**: Usually shared across environments or replicated per subscription/account

This exception is necessary because you cannot use Terragrunt's remote state configuration until the state storage itself exists.

#### Example Terragrunt Configuration

```hcl
# environment/module.hcl
terraform {
  source = "git@github.com:padok-team/terraform-aws-environment.git?ref=v1.0.0"
}
```

```hcl
# common.hcl
inputs = {
  tenant_id       = "12345678-1234-1234-1234-123456789012"
  database_size   = "db.t3.small"
  backup_enabled  = true
}
```

```hcl
# environment/dev/inputs.hcl
inputs = {
  # Override for dev environment
  database_size  = "db.t3.micro"
  backup_enabled = false
  environment    = "dev"
}
```

```hcl
# environment/dev/terragrunt.hcl
include "root" {
  path = find_in_parent_folders()
}

include "envcommon" {
  path = "${dirname(find_in_parent_folders())}/environment/module.hcl"
}

include "inputs" {
  path = "inputs.hcl"
}
```

### Layer Guidelines

- A layer = a directory where you run `terraform apply/destroy` = one terraform state
- Layer size indicators (too large when):
  - Terraform plan/apply refresh time becomes slow
  - High number of collaborators (causes state locking incidents)
  - Multiple teams ownership conflicts
- Only one team should own a layer

### Module Creation Rules

Create modules for:

- ✅ Creating a resource multiple times (e.g., network, databases)
- ✅ Hiding complexity to create a resource (e.g., EKS cluster)
- ❌ NOT for passing through another module (anti-pattern)

## File and Folder Naming

### Files

- `_settings.tf`: Terraform configuration (required_version, providers, backend)
- `output.tf`: Layer/module interface with the outside world
- Use `reader_friendly_name.tf` for resource files (e.g., `database.tf`, `network.tf`)
- **Avoid `main.tf`** - use descriptive names instead

### Folders

- Root folders: Named by project need (e.g., `environment`, `application`)
- Layer folders: Named by subproject need (e.g., `production`, `dev`, `app_1`)
- Module folders: Named after what they define

## Naming Conventions

### Use snake_case Only

- All resource and variable names must use `snake_case`
- Allowed characters: lowercase alphanumeric with `_` separators

### Resource Naming

- Avoid stuttering: `aws_route_table.public` ✅, `aws_route_table.public_route_table` ❌
- Use `this` for single resources in modules: `google_storage_bucket.this`
- Use `these` for multiple resources: `google_storage_bucket.these`
- Use plural names for `for_each` loops: `module.buckets`

### Variables

- Collection variables (sets, lists, maps) must be plural
- Map variable names should reflect key-value relationship
- Always provide descriptions, even if obvious
- Reuse argument names from Terraform documentation when possible

### Outputs

- Lists must have plural names
- Maps should obviously relate to key/value pairs
- Always provide descriptions
- Use `this` for main resource output in modules

## Versioning Standards

### Environment Layers (Be Unambiguous)

- **Terraform >= 1.0**: Use exact versions `= 1.6.0`
- **Providers**: Use exact versions for deterministic behavior `= 4.0.0`
- **Lock files**: Always commit `.terraform.lock.hcl` files

### Modules (Be Flexible)

- **Terraform CLI**: Use flexible versions `~> 1.0`
- **Providers**: Use flexible versions `~> 4.0`
- **Remote modules**: Always pin to specific versions/tags

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.2"  # ✅ Specific version
}

module "vpc" {
  source = "git@github.com:padok-team/terraform-aws-network.git?ref=v0.2.0"  # ✅ Specific tag
}
```

## Red Flags to Avoid

### Terraform Workspaces

- **Never use** `terraform workspace` commands
- Creates environment isolation issues and confusion

### One-liners

- Avoid complex nested function wrapping
- Split complexity into intermediate values
- Prioritize readability over cleverness

### TFvars Files

- Prefer `locals` over `tfvars` with variable declarations
- Reduces boilerplate and complexity

### Monolayer Repositories

- Avoid putting all resources in root directory
- Use proper layer segmentation

### Shell Scripts in IaC

- Keep shell scripts in separate folders/repositories
- Don't mix scripts with `.tf` files

### Provisioners

- Avoid Kubernetes and Helm providers in Terraform
- Use proper provisioning tools (Ansible) for deployment tasks

## Internal Library Usage

When working with cloud providers, prefer our internal libraries, they have starter ready to go.

- **Azure**: `padok-team/azure-library`
- **AWS**: `padok-team/aws-library`
- **GCP**: `padok-team/gcp-library`

## Code Quality & Security

### Required Tools Integration

- **tflint**: Use for Terraform linting with provider-specific rules
- **checkov**: Static security analysis for infrastructure code
- **terraform fmt**: Format code consistently
- **terragrunt hclfmt**: Format Terragrunt files
- **tfautomv**: Automate refactoring with moved blocks

### Pre-commit Configuration

```yaml
repos:
  - repo: https://github.com/antonbabenko/pre-commit-terraform
    rev: v1.77.0
    hooks:
      - id: terraform_fmt
      - id: terragrunt_fmt
      - id: terraform_checkov
      - id: terraform_providers_lock
```

## Security Best Practices

### Secrets Management

- **Never** use plain text secrets in code
- Use secure access to Terraform state
- Leverage cloud provider secret management services
- Use data sources to reference existing secrets

### Provider Configuration

- Don't specify provider blocks in modules (let calling code handle it)
- Use provider aliases for multi-region deployments
- Follow principle of least privilege for service accounts

## Refactoring Guidelines

### When to Refactor

- Plan/apply time becomes too slow
- Too many collaborators on single layer
- Multiple teams need different ownership
- Module interfaces become too complex

### Refactoring Process

1. Make structural changes only (no configuration changes)
2. Run `tfautomv` to generate move blocks
3. Apply moves (should result in empty plan)
4. Then make configuration changes in separate operation

### Use tfautomv for:

- ✅ Renaming resources
- ✅ Moving between modules
- ✅ Converting to for_each loops
- ✅ Module restructuring
- ❌ Changing resource configuration simultaneously

## Cross-Layer References

### Preferred Methods

1. **Data sources**: Query existing resources by tags/names
2. **Remote state**: Access outputs from other layers
3. **String values**: ❌ Avoid hardcoded strings

### Data Source Pattern

```hcl
data "azurerm_resource_group" "network" {
  name = "${var.project}-${var.environment}-network"
}

data "azurerm_subnet" "private" {
  name                 = "private"
  resource_group_name  = data.azurerm_resource_group.network.name
  virtual_network_name = "${var.project}-${var.environment}-vnet"
}
```

## Terraform State Management

### Standards for Layers

- Can rebuild infrastructure without `-target` flag
- No cyclical dependencies between resources
- Use data blocks only in layers (not modules)

### Standards for Modules

- Module dependencies provided by caller
- Use remote modules to avoid repetition
- Don't create modules just to pass through to other modules

## Additional Recommendations

### Optional Attributes

- Safe to use since Terraform 1.3.0 (generally available)
- Helps reduce boilerplate in complex object variables

### Iteration Patterns

- Use `for_each` over `count` for better resource addressing
- Use `for` expressions for data transformation
- Consider `dynamic` blocks for conditional resource configuration

### Documentation

- Always include README.md in modules
- Use terraform-docs for automatic documentation generation
- Document complex locals and data transformations

## Testing & Validation

### Validation Process

1. Run `terraform plan` to verify no unintended changes
2. Use `terraform validate` for syntax checking
3. Apply security scanning with checkov
4. Test in isolated environment first

### CI/CD Integration

- Run pre-commit hooks in CI pipeline
- Generate and validate Terraform plans
- Use environment-specific variable files
- Implement approval workflows for production changes

## Code Review Guidelines

### Review Scope

Code reviews must cover both Terraform (.tf) and Terragrunt (.hcl) files to ensure compliance with all IaC Guild standards and maintain code quality across the infrastructure codebase.

### Core Review Principles

- **Standards Compliance**: Verify adherence to all naming conventions, versioning, and structural guidelines
- **Security Focus**: Identify potential security vulnerabilities and misconfigurations
- **Maintainability**: Ensure code is readable, well-documented, and follows established patterns
- **Business Alignment**: Confirm changes align with the 3 Gold Standards for infrastructure

### Review Checklist

#### 🏗️ **Project Structure & Patterns**

- [ ] **Pattern Consistency**: Changes follow WYSIWYG or Context pattern appropriately
- [ ] **Layer Design**: New layers reflect genuine business needs
- [ ] **Team Ownership**: Clear ownership boundaries maintained
- [ ] **Folder Structure**: Directories follow naming conventions (environment, application, etc.)
- [ ] **File Organization**: Files are properly organized and named descriptively

#### 📝 **Naming Conventions**

- [ ] **snake_case Usage**: All resources, variables, and outputs use snake_case
- [ ] **Resource Naming**: No stuttering (e.g., `aws_route_table.public` not `aws_route_table.public_route_table`)
- [ ] **Module Resources**: Single resources use `this`, multiple use `these`
- [ ] **Variable Naming**: Collections are plural, maps reflect key-value relationships
- [ ] **File Naming**: Use descriptive names, avoid `main.tf`
- [ ] **Descriptions**: All variables and outputs have meaningful descriptions

#### 🔧 **Module Standards**

- [ ] **Module Purpose**: Created for valid reasons (reuse or complexity hiding)
- [ ] **Anti-patterns**: No pass-through modules
- [ ] **Documentation**: README.md present and up-to-date
- [ ] **Interface Design**: Clean input/output interfaces
- [ ] **Remote vs Local**: Appropriate choice between local and remote modules

#### 🔢 **Versioning & Dependencies**

- [ ] **Layer Versioning**: Exact versions used for Terraform and providers (`= 1.6.0`)
- [ ] **Module Versioning**: Flexible versions used in modules (`~> 1.0`)
- [ ] **Lock Files**: `.terraform.lock.hcl` files committed for layers
- [ ] **Remote Modules**: Properly pinned to specific versions/tags
- [ ] **Provider Versions**: Deterministic provider versions specified

#### 🛡️ **Security & Code Quality**

- [ ] **Secrets Management**: No plain text secrets in code
- [ ] **Data Sources**: Secrets referenced via data sources
- [ ] **Provider Configuration**: No provider blocks in modules
- [ ] **Resource Dependencies**: Clear and logical dependency chains
- [ ] **State Management**: No cyclical dependencies

#### 🚩 **Red Flags & Anti-patterns**

- [ ] **Workspace Usage**: No `terraform workspace` commands
- [ ] **One-liners**: Complex logic split into readable chunks
- [ ] **TFvars**: Prefer `locals` over `tfvars` with variable declarations
- [ ] **Shell Scripts**: No scripts mixed with `.tf` files
- [ ] **Provisioners**: No Kubernetes/Helm providers in Terraform
- [ ] **Monolayer**: Proper layer segmentation maintained

#### 🔗 **Cross-Layer References**

- [ ] **Data Sources**: Preferred over hardcoded strings
- [ ] **Remote State**: Proper implementation for layer communication
- [ ] **Hardcoded Values**: No hardcoded resource IDs or names
- [ ] **Dependencies**: Clear and documented cross-layer dependencies

#### 📋 **Terragrunt-Specific Checks**

- [ ] **Context Pattern**: Proper use of `common.hcl`, `_settings.hcl`, `module.hcl`
- [ ] **Input Heaven**: Inputs properly distributed through tree structure
- [ ] **Include Blocks**: Correct include hierarchy and references
- [ ] **Uniqueness**: Each layer defines one and only one module
- [ ] **DRY Principle**: No unnecessary duplication across environments

#### 📚 **Documentation & Comments**

- [ ] **Code Comments**: Complex logic explained with comments
- [ ] **Change Rationale**: PR description explains the why, not just the what
- [ ] **Module Documentation**: terraform-docs generated documentation
- [ ] **Variable Descriptions**: Clear descriptions for all inputs and outputs

### Review Process

#### **Before Review**

1. **Automated Checks**: Ensure all pre-commit hooks pass (terraform fmt, tflint, checkov)
2. **Plan Generation**: Review terraform plan output for intended changes
3. **Self-Review**: Author performs self-review against this checklist

#### **During Review**

1. **Standards Verification**: Use checklist to verify compliance systematically
2. **Security Assessment**: Focus on security implications of changes
3. **Impact Analysis**: Consider effects on other layers and teams
4. **Documentation Review**: Ensure adequate documentation and comments

#### **Review Outcomes**

- **Approve**: All standards met, no blocking issues
- **Request Changes**: Standards violations or security concerns identified
- **Comment**: Suggestions for improvement without blocking merge

### Change-Specific Guidelines

#### **New Module Reviews**

- [ ] Module serves a clear, reusable purpose
- [ ] Interface design is clean and intuitive
- [ ] Documentation includes usage examples
- [ ] Follows internal library patterns where applicable

#### **Environment Changes**

- [ ] Changes applied consistently across environments where appropriate
- [ ] Production changes have extra scrutiny
- [ ] Rollback plan considered for significant changes

#### **Refactoring Reviews**

- [ ] Structural changes separated from configuration changes
- [ ] `tfautomv` used appropriately for resource moves
- [ ] No unintended infrastructure changes in plan

#### **Version Updates**

- [ ] Breaking changes identified and documented
- [ ] Upgrade path clear and tested
- [ ] Lock files updated appropriately

### Common Review Comments

#### **Standards Violations**

```markdown
❌ **Naming Convention**: Resource `aws_route_table.public_route_table` violates anti-stuttering rule.
✅ **Suggested Fix**: Rename to `aws_route_table.public`
```

#### **Security Concerns**

```markdown
🔒 **Security**: Hard-coded database password detected in line 23.
✅ **Suggested Fix**: Use data source to reference AWS Secrets Manager secret
```

#### **Module Issues**

```markdown
🔧 **Module Design**: This module appears to be a pass-through for another module.
✅ **Suggested Fix**: Consider using the upstream module directly or adding genuine abstraction value
```

#### **Documentation**

```markdown
📝 **Documentation**: Variable `subnet_ids` lacks description.
✅ **Suggested Fix**: Add description explaining expected format and usage
```

---

_These guidelines help maintain consistency, readability, and maintainability across our IaC codebase. They represent recommendations based on our experience - adapt them to your specific project needs while maintaining the core principles._
