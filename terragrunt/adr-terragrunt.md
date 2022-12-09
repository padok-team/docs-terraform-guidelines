# Terragrunt ADR

- [Terragrunt ADR](#terragrunt-adr)
  - [Issue](#issue)
  - [Decision](#decision)
  - [Status](#status)
  - [Assumptions](#assumptions)
  - [Constraints](#constraints)
  - [Positions](#positions)
    - [Terraform](#terraform)
    - [Terragrunt](#terragrunt)
    - [Terramate](#terramate)
    - [Pulumi](#pulumi)
  - [Argument](#argument)
  - [Implications](#implications)

## Issue

We have been using Terraform to configure our infrastructures at scale. We started facing issues maintaning and onboarding new comers to a code base.
The [WYSIWYG pattern](../wysiwg_patterns.md) helped keep common pratices between code bases it has limitation : you need to duplicate code (Modules, resources and data) in order to scale your infrastructure.
This increases complexity when creating new infrastructure because you need to understand every parametre of your modules. Thus module complexity tend to increase and refactoring becomes harder.

## Decision

We have decided to adopt Terragrunt for creating infrastructre at scale.

## Status

Decided

## Assumptions

- We have fully adopted Terraform and continue to believe that it is the best tool to configure infrastructure.
- Configuring your cloud infrastructure with code is the best way for SRE to maintain majority of an infrastructure (Other possibility being cloud policies or orchestration tools like ansible)

## Constraints

- The solution has to be compatible with Terraform and should be easily accesible.
- The solution keep our code base DRY (Don't repeat yourself)

## Positions

### Terraform

Vanilla [Terraform](https://www.terraform.io/) does not allow you to keep your code DRY. The [Deepmerge module from cloudpose](https://github.com/cloudposse/terraform-yaml-config/tree/0.2.0/modules/deepmerge) can help you merge inputs but it's more of a hack than anything else.

### Terragrunt

The [contexte pattern](../context_pattern.md) explains ways around the issue and how [Terragrunt](https://terragrunt.gruntwork.io/) help you have a DRY code base.
It solves the issue with :

- Distributing inputs from a module between multiple files
- Ease of refactoring because one layer equals one module

### Terramate

[Terramate](https://github.com/mineiros-io/terramate) looks like Terragrunt but has less adoption inside the community.

### Pulumi

We have looked into [Pulumi](https://www.pulumi.com/) because it uses common language like GO, typescript or Python, but it does not help solve the issue. Furthemore the solution is still a bit young and still base most of its provider on Terraform provider.

## Argument

We chose Terragarunt because:

- It's additive to our Terraform code and knowledge
- The learning curves is quite fast.
- The cost does not changes sinces it's opensource.
- **It enforces a DRY code**

## Implications

- Create library starter with Terragrunt
- Create a decison matrix to choose between Terraform and Terragrunt
