# Dont's / Red Flags

## Workspaces

It's the usage of[`terraform workspace`](https://developer.hashicorp.com/terraform/language/state/workspaces) commands.

### Drawbacks

- One backend for all environments, so no isolation between environments.
- One version for all environments, so no immutable infrastructure.
- You cannot know the number of workspaces (or environments) just by reading the code.
- Brings confusion in which workspace the `terraform apply` command is performed, and so, can induce mistaking prod and non-prod environments.
- You're adding options to your command line.

### Solutions

A solution can be to have some code duplication for each environment.

## One-liners

It consists in wrapping functions in functions in order to parse inputs.

Example

```hcl
locals {
   read_replica_private_ip = var.enable_ha ? lookup(zipmap([for addr in module.postgres.replicas_instance_first_ip_addresses.0 : values(addr).2],
   [for addr in module.postgres.replicas_instance_first_ip_addresses.0 : values(addr).0]), "PRIVATE", null) : ""
   read_replica_public_ip = var.enable_ha ? lookup(zipmap([for addr in module.postgres.replicas_instance_first_ip_addresses.0 : values(addr).2],
   [for addr in module.postgres.replicas_instance_first_ip_addresses.0 : values(addr).0]), "PRIMARY", null) : ""
 }
```

### Drawbacks

It is very tempting to write one-liners that perfectly parses maps or lists to other wanted structures.
But, wraping functions in functions brings complexity that will be painfull to understand while maintaining code. It quickly becomes technical debt.

### Solutions

A solution can be to same intermediate values of the parsed input, so that we split the complexity.
Another one would be to rely on outputs from subsequent modules or resources.

## Shell scripts in IaC codebase

Shell scripts or other files within layers or modules aka folders with `.tf` files are just trash.
Please store them in another dedicated folder or, even better, another repo.

### Drawbacks

- You may rely on scripts to apply and forget about how init and apply are done.
- They'll be duplicated when you duplicate the folder with copy-paste, creating more junk.

## Provisionning/Provisionners

This is about drawing the line between **Providers** and **Provisionners**.

Providers like terraform are used for (but not limited to):

- create infrastructure parts
- define network rules
- handle IAM

Provisionners like Ansible are used for (but not limited to):

- deploy applications
- change configurations

So, in terrafom, don't use the following providers: Kubernetes; helm.

### Drawbacks

- Loose idempotence.

## TFvars

We prefer to use locals that do not require variable declarations to be used.

So instead of this

```hcl
# dev.tfvars
env = dev
project = my_project
```

```hcl
# main.tf
variable "env" {}
variable "project" {}

resource "null" "this" {
  project = var.project
  env = var.env
}
```

We have this

```hcl
# main.tf
locals {
  env = dev
  project = my_project
}

resource "null" "this" {
  project = local.project
  env = local.env
}
```

## Monolayer repos

These are projects that look like this (every terraform files at the root of the repository).

```plaintext
.
├── README.md
├── backend.tf
├── network.tf
├── iam.tf
├── app1_rds.tf
├── app1_S3.tf
└── provider.tf
```

### Drawbacks

- Collaboration is deeply impacted since we have a mono state file.
- `terraform plan` will take longer and longer each time we add a resource.
- The [blast radius](https://learn.microsoft.com/azure/architecture/framework/resiliency/chaos-engineering#blast-radius) is important since all resources are close to each other.

## Multi-layer repos

This are projects that look like this (we store files in a tree arborescence with a depth of 1).

```plaintext
.
├── README.md
└── cloudrun
    ├── README.md
    ├── backend.tf
    ├── main.tf
    ├── provider.tf
    ├── tfvars
    │   ├── integration.tfvars
    │   ├── production.tfvars
    │   ├── staging.tfvars
    │   └── testing.tfvars
    └── variables.tf
```

### Advantages

This type of repo brings flexibity alongside collaboration improvments.

### Drawbacks

- Too much layers can cause a higher maintainability cost.
- We cannot easily use outputs of one layer in another.
- Some non-selectives CI may run `terraform plan` over all layers.

## Using provider block in modules

When you create a terraform module, you may be tempted to add a `provider` block in your code. Using this pattern, you will have a structure looking like this.

```bash
.
├── environment
│   ├── core.tf
│   └── _settings.tf
└── modules
    └── core
        ├── resource_group.tf
        └── _settings.tf
```

**module/core**

```hcl
# _settings.tf
terraform {

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~>2.89"
    }
  }
}

provider {}

# resource_group.tf
resource "azurerm_resource_group" "example" {
  name     = "example"
  location = "west europe"
}
```

**environment**

```hcl
# _settings.tf
terraform {

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "=2.89"
    }
  }
}

provider {}

# core.tf
module "core" {
  source = "../modules/core"
}
```

### Advantages

Your module will be easier to test as a standalone and you won't need any other configuration.

### Drawbacks

Terraform manages its code, structure and state very specificly. Each resource will attempt to match its neerest `provider` to instanciate itself and the resource will keep it _reference provider_.
If you use a module having such a block in its code, when you will want to move or destroy it, terraform will have conflict because it will not find this _reference_ anymore and won't be able to reconstruct its state.

Prefer to declare your `provider` blocks in your layers.

More information in [providers within modules](https://developer.hashicorp.com/terraform/language/modules/develop/providers) documentation.

## Using type `any` in variables

The type `any` is a special type that can be used in variables. The value of a variable with type `any` can be of any type supported by terraform.

```hcl
variable "my_variable" {
  type = any
}
```

or

```hcl
variable "my_other_variable" {
  type = object({
    special_property = any
  })
}
```

### Advantages

Using type `any` in variables saves time during development because you don't have to be specific about the type of value you want to use.

### Drawbacks

Using type `any` outside of development is a bad practice for two reasons :

- The users of your module will not know what type of value they should use. They will have to guess it by looking at your code.
- Your code will be subject to errors at runtime if the consumers of your `any` typed variable try to use it in a way that the user didn't expect.

Using type `any` during development is fine, but you should always replace it with the correct expected type before using your module in production.

## Data sources depending on values only known at apply time

A `data` source is read by Terraform to fetch information about objects that already exist outside of the current configuration. When all the arguments of a `data` block are known at plan time (static values, variables, locals...), Terraform reads it **during the plan** and its result is available right away.

However, when one of its arguments references a value that is only known **after** apply — typically an attribute of a managed `resource` created in the same configuration — Terraform cannot read the data source during the plan and defers the read to the apply phase.

```hcl
resource "aws_instance" "this" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
}

# `instance_id` depends on `aws_instance.this.id`, which is only known after apply
data "aws_instance" "this" {
  instance_id = aws_instance.this.id
}
```

### Drawbacks

- The data source shows up as a `(known after apply)` read in your plan, adding noise to the output.
- Your plan no longer reflects the real set of changes: you cannot trust it to review exactly what will happen before applying.
- Anything that consumes the deferred data source is in turn marked as `(known after apply)`, which cascades and can hide actual changes.

### Solutions

- Reference the managed resource attribute directly instead of reading it back through a data source. If you created the resource, its attributes are already available on the `resource` block.

```hcl
resource "aws_instance" "this" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
}

# Use the resource attribute directly, no data source needed
output "instance_arn" {
  value = aws_instance.this.arn
}
```

- Use `data` blocks only to fetch objects that are **not** managed by the current configuration, in line with the standard [you use data blocks only in layers](../README.md#the-standards-for-layers).

More information in the [data source behavior](https://developer.hashicorp.com/terraform/language/data-sources#data-source-behavior) documentation.
