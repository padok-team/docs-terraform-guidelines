# IaC Guidelines

Guidelines to work with Terraform and Terragrunt

- [IaC Guidelines](#iac-guidelines)
  - [🎯 Our purpose](#-our-purpose)
  - [Why IaC ?](#why-iac-)
  - [🌟 Standards 🌟](#-standards-)
    - [The 3 gold standards for your infrastructure codebase](#the-3-gold-standards-for-your-infrastructure-codebase)
    - [The standards for layers](#the-standards-for-layers)
    - [The standards for modules](#the-standards-for-modules)
    - [The standard naming convention](#the-standard-naming-convention)
  - [🚀 Guidelines](#-guidelines)
    - [🛗 Patterns](#-patterns)
    - [🔄 Lifecycle operations](#-lifecycle-operations)
    - [🎓 Standards](#-standards)
      - [Terraform](#terraform)
      - [Terragrunt](#terragrunt)
    - [🚩 Red flags](#-red-flags)
    - [🛠️ Tooling](#️-tooling)
  - [License](#license)

## 🎯 Our purpose

This documentation is provided by the Theodo Cloud IaC Guild. Its purpose is to present guidelines and best practices about terraform/terragrunt development.

You do not have to see these guidelines as an absolute truth, but more as a proposition to answer frequently asked questions and avoid issues you will often face in your projects.

## Why IaC ?

Infrastructure as Code (IaC) is a key practice in modern software development that allows teams to manage and provision infrastructure through code, rather than manual processes. This approach brings several benefits:

- **Consistency**: IaC ensures that infrastructure is provisioned consistently, reducing the risk of human error and configuration drift.
- **Version Control**: Infrastructure code can be versioned, allowing teams to track changes,
- **Automation**: IaC enables automation of infrastructure provisioning, scaling, and management, leading to faster deployments and reduced operational overhead.
- **Collaboration**: Teams can collaborate on infrastructure changes using the same tools and processes they use for application code, fostering better communication and alignment.
- **Scalability**: IaC allows for easy scaling of infrastructure resources, enabling teams to adapt to changing demands quickly.

## 🌟 Standards 🌟

### The 3 gold standards for your infrastructure codebase

- Your layer design reflects a business need
- Only one team contributes has ownership of a layer (you have a clear vision of contribution & ownership)
- Your state refresh time is acceptable for a contribution purpose

### The standards for layers

- You can rebuild your infrastructure without the `-target` flag
- You do not have any cyclical dependencies between resources
- You use data blocks only in layers

### The standards for modules

- A module should be created
  - To abstract complexity
  - If you have to reuse it frequently (More than twice)
- You should not create a module to be a flat pass between resources
- Your module’s dependencies are provided by the caller
- You use remote modules to avoid repetition between your different projects

### The standard naming convention

- The name of the file or folder should reflect what you are creating
- Adapt your codebase to your client needs and standards
- Avoid stuttering when naming Terraform resources.
- Use snake_case convention

We recommend everyone to follow the [WYSIWYG pattern](terraform/wysiwg_patterns.md) for vanilla terraform and the [Context pattern](terragrunt/context_pattern.md) for Terragrunt.

## 🚀 Guidelines

You’ll find below details of the standards to follow when working with Terraform and Terragrunt.

### 🛗 Patterns

> Reusable solution to a commonly occurring problem within a given context

- [WYSIWYG pattern](terraform/wysiwg_patterns.md)
- [Context pattern aka the Terragrunt implementation](terragrunt/context_pattern.md)

### 🔄 Lifecycle operations

- [Choosing the right IaC structure](operations/standard-best-iac-structure.md)
- [Refactoring](operations/refactoring.md)

### 🎓 Standards

> Standards help to avoid waste and ensure that we deliver value

#### Terraform

- [Distant values references](terraform/refering_to_resources_from_other_layers.md)
- [Versioning](terraform/terraform_versioning.md)
- [Naming](terraform/terraform_naming.md)
- [Pre-commits](terraform/pre-commits.md)
- [Iterate on your resources](terraform/iterate_on_your_resources.md)
- [Optional attributes](terraform/optional_attributes.md)
- [Handling secrets](terraform/handling_secrets.md)

#### Terragrunt

- [Why Terragrunt (ADR)](terragrunt/adr-terragrunt.md)
- [Terragrunt guidelines](terragrunt/context_pattern.md)
- [Distant values references](./terragrunt/refering_to_resources_from_other_layers.md)

### 🚩 Red flags

> Red flag is something that you must pay attention about. This is an advice or recommendation, not a requirement.

- [Dont’s](terraform/donts.md)
- [Child modules depth limit](terraform/child_modules_depth_limit.md)

### 🛠️ Tooling

- [Useful tooling for Terragrunt/Terraform](tooling/README.md)
- [Terraform pre-commits](terraform/pre-commits.md)
- [TFLint configuration](tooling/tflint.md)

## License

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
