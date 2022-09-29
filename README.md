# Terraform Guidelines

Guidelines to work with Terraform

## 🎯 Our purpose

This documentation is provided by the Padok IaC Guild. Its purpose is to present some guidelines and best practices about terraform development. 

You do not have to see these guidelines as an absolute truth but more as a proposition to answer frequently asked questions and avoid issues you will often face in your projects.

Our goal is to guide you in this serie of topics:

**1. Layer/code organisation**
- Your layer corresponds to a unique terraform state
- Your code design reflects your business need
- Only one team contributes on a layer (you have a clear vision of contribution & ownership)
- Your state refresh time is acceptable for a contribution purpose

**2. Communication between your layers**
- Your can rebuild your infrastructure without the `-target` flag
- You do not have any cyclical dependencies between resources
- You use data blocks only in layers

**3. Use of modules**
- Your module's dependencies are provided by the caller 
- Your module abtracts code complexity
- Your module integrates custom logic / guidelines
- Your module is not a resource flat pass
- Your module is used in different layers
- You use remote modules to avoid repetition between your different projects 

**4. Naming convention**
- You follow the WYSIWYG pattern
- You adapt your codebase to your client needs and standards
- You avoid stuttering
- You use snake_case convention

## 🚀 Guidelines

You'll find bellow standards to follow when working with Terraform.

- [Patterns](patterns.md)
- [Versioning](terraform_versioning.md)
- [Naming](terraform_naming.md)
- [Pre-commits](pre-commits.md)
- [Distant values references](refering_to_resources_from_other_layers.md)

## 🚩 Red flags

Red flag is something that you must pay attention about. This is an advice or recommendation not a requirement.

- [Don'ts](donts.md)
- [Optional attributes (experimental feature)](optional-attributes.md)
- [Child modules depth limit](child_modules_depth_limit.md)
