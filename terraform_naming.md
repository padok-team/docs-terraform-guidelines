# Terraform Naming Guidelines

## `use_snake_case_0nly`

All resource or variable names in Terraform code are written in `snake_case`. 
This means allowed characters are lowercase alphanumeric characters, with `_`
separators when needed.

Why ?
- it is the standard in the terraform ecosystem
- not following the standard can cause confusion in the team and slow down 
  onboarding
- these guidelines are what is recommended by the [Terraform Best Practices](https://www.terraform-best-practices.com/naming) site (*do not hesitate to check this website out!*)

> Note : Beware that actual cloud resources often have restrictions in allowed names. Some resources, for example, can't contain dashes, some must be camel-cased. The conventions in this document refer to Terraform names themselves.


## Resource and/or data source naming

Follow the DRY principle (Do not Repeat Yourself) when naming Terraform
resources. Examples : 

```hcl
resource "aws_route_table" "public" {}                 # <- 🟢 no repetitions 
resource "aws_route_table" "public_route_table" {}     # <- 🔴 repetitions
resource "aws_route_table" "public_aws_route_table" {} # <- 🔴 many repetitions
```

In the context of a module, a resource whose name is self-explanatory should
directly be named `this`. Example taken from the [terraform-google-staticfrontend](https://github.com/padok-team/terraform-google-staticfrontend)
module in the Padok library : 
```hcl
resource "google_storage_bucket" "this" {
  name     = var.name
  ...
}
```

Resource and module instance names should always be singular.

## Variables


Variable names for **lists** or **maps** must be plural.


Avoid repeating yourself within a variable's name. Example :
```hcl
variable "s3_storage_bucket_name" # <- 🔴 too precise
variable "bucket"                 # <- 🔴 not precise enough
variable "bucket_name"            # <- 🟢 seems like a good compromise
```

Always provide a variable's description, even if it seems obvious to you.

```hcl
variable "tags" {
  description = "A mapping of tags to assign to the resource."
  type        = map(string)
  default = {
    terraform = "true",
  }
}
```

Do not reinvent the wheel when naming variables or writing descriptions. If a
variable is consumed by a resource or module directly, maybe reuse the variable
name from the "**Argument Reference**" section in the resource's (or module's)
documentation.


## Module outputs

Naming module outputs properly is important for them to be consistent through your
codebase, and for them to be understandable outside of their immediate context.
From a user's perspective, it should be trivial what type each attribute has.

If the output is a **list** or a **map**, its name must be plural.

Always provide an output's description, even if it seems obvious to you.

Example taken from the [terraform-azurerm-kubernetes](https://github.com/padok-team/terraform-azurerm-kubernetes)
module from the Padok Library.

```bash
output "this" {
  description = "The AKS resource."
  value       = data.azurerm_kubernetes_cluster.this
}
```

Example taken from the [terraform-azurerm-function_app-blueprint](https://github.com/padok-team/terraform-azurerm-function_app-blueprint)
module from the Padok Library. It instanciates a Function App, a Storage Account
and an App Insights.

```bash
output "this" {
  description = "The Function App resource."
  value       = module.azurerm_function_app.this
}

output "app_insights" {
  description = "The Application Insights resource."
  value       = module.azurerm_application_insights.this
}

output "storage_account" {
  description = "The Storage Account resource."
  value       = module.azurerm_storage_account.this
}
```
