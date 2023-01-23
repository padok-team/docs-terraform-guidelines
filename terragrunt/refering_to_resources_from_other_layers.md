# Refering to resources from other layers

As terragrunt purpose is to have a better management of your layers dependencies, it includes some features simplifying remote state references.

## TL;DR

> Terragrunt includes `dependency` features to get remote layer informations.

## Using `dependency` feature

In terragrunt, you can refer to another layer of your infrastructure using `dependency` feature.
By using this feature:

- You will be able to access a remote state output
- You can create a dependency tree between your layers
- Your apply will fail if outputs of your dependency layer are not available
- ⚠️ It **will not** launch an apply on the dependency layers on its own and you can end up with outdated outputs

Applying a layer using a dependency will enforce the parent layer apply if needed, and allow you to make sure your infrastructure has no drift.

If you want highly consistent outputs and layers dependencies, you may want to run `terragrunt run-all` at your project root.

## To go further

Feel free to read the [terragrunt documentation](https://terragrunt.gruntwork.io/docs/reference/config-blocks-and-attributes/#dependency) to investigate deeper configurations for dependencies.
For exemple, You will be able to define mocked outputs and dependency skip in your code to simplify your development and deployment.
