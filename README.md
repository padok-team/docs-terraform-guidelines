# IAC Guidelines

Guidelines to work with Terraform and Terragrunt

- [IAC Guidelines](#iac-guidelines)
  - [🎯 Our purpose](#-our-purpose)
    - [The 3 standards for a layer/code](#the-3-standards-for-a-layercode)
    - [The standard for communicating between your layers](#the-standard-for-communicating-between-your-layers)
    - [The standard use of modules](#the-standard-use-of-modules)
    - [The standard naming convention](#the-standard-naming-convention)
  - [🚀 Guidelines](#-guidelines)
    - [🛗 Patterns](#-patterns)
    - [🎓 Standards](#-standards)
      - [Terraform](#terraform)
      - [Terragrunt](#terragrunt)
    - [🚩 Red flags](#-red-flags)
    - [🛠️ Tooling](#️-tooling)
  - [License](#license)

## 🎯 Our purpose

This documentation is provided by the Padok IaC Guild. Its purpose is to present guidelines and best practices about terraform development.

You do not have to see these guidelines as an absolute truth, but more as a proposition to answer frequently asked questions and avoid issues you will often face in your projects.

Here are the key learning :

### The 3 standards for a layer/code

- Your layer design reflects a business need
- Only one team contributes has ownership of a layer (you have a clear vision of contribution & ownership)
- Your state refresh time is acceptable for a contribution purpose

### The standard for communicating between your layers

- You can rebuild your infrastructure without the `-target` flag
- You do not have any cyclical dependencies between resources
- You use data blocks only in layers

### The standard use of modules

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

We recommend everyone to follow the [WYSIWYG pattern](terraform/wysiwg_patterns.md) for vanilla terraform.

## 🚀 Guidelines

You’ll find below details of the standards to follow when working with Terraform and Terragrunt.

### 🛗 Patterns

> Reusable solution to a commonly occurring problem within a given context

- [WYSIWYG pattern](terraform/wysiwg_patterns.md)
- [Context pattern aka the Terragrunt implementation](terragrunt/context_pattern.md)

### 🎓 Standards

> Standards help to avoid waste and ensure that we deliver value

#### Terraform

- [Distant values references](terraform/refering_to_resources_from_other_layers.md)
- [Versioning](terraform/terraform_versioning.md)
- [Naming](terraform/terraform_naming.md)
- [Pre-commits](terraform/pre-commits.md)
- [Iterate on your resources](terraform/iterate_on_your_resources.md)
- [Optional attributes](terraform/optional-attributes.md)

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

## License

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
