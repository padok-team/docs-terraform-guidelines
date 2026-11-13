# Unused inputs

Terragrunt passes the `inputs` of a layer to the Terraform module it calls. Nothing checks that those inputs match the variables the module actually declares.

## TL;DR

> Every input of a layer must match a variable declared in the module it calls.
> Remove or rename the inputs that don't.

## Do not set inputs that the module does not declare

Terragrunt exposes the `inputs` as `TF_VAR_*` environment variables. Terraform **silently ignores** the ones that have no matching `variable`: no error, no warning.

### Why?

- **Typos go unnoticed**
  - A misspelled input is ignored, and the real variable keeps its default value or fails far from the cause.
- **Dead code accumulates**
  - When a variable is removed from a module, the inputs that fed it stay in the layers. Readers assume they still have an effect.
- **Inherited inputs are hard to track**
  - With the [Context pattern](context_pattern.md), inputs come from `root.hcl`, `module.hcl` and parent `inputs.hcl` files. An obsolete input high in the tree is hard to spot and applies to every layer below it.

### Recommendation

Declare the variable in the module, and set the input with the exact same name.

Ok :+1:

```hcl
# modules/bastion/variables.tf
variable "subnet_ids" {
  type = list(string)
}

# layers/bastion/dev/inputs.hcl
inputs = {
  subnet_ids = ["subnet-0123456789"]
}
```

Not recommended ❌

```hcl
# modules/bastion/variables.tf
variable "subnet_ids" {
  type = list(string)
}

# layers/bastion/dev/inputs.hcl
inputs = {
  subnet_id = ["subnet-0123456789"] # typo: there is no variable "subnet_id"
}
```

## How to fix it

- **Typo**: rename the input to match the variable.
- **Variable removed from the module**: delete the input from the layer, and from its parents if no other layer uses it.
- **Variable missing from the module**: declare it in the module if the module really needs it.
- **Input defined too high in the tree**: an input in `root.hcl` is inherited by every layer, including those whose module does not declare it. Move it down to the `module.hcl` or `inputs.hcl` of the layers that need it (see [Refactoring with Terragrunt](../operations/refactoring.md#refactoring-with-terragrunt)).
