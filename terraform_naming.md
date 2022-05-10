# Terraform Naming Guidelines

## `use_snake_case_0nly`

All resource or variable names in Terraform code must be written in `snake_case`. 
This means allowed characters are lowercase alphanumeric characters, with `_`
separators when needed.

Why ?
- it is the standard in the terraform ecosystem
- not following the standard can cause confusion in the team and slow down 
  onboarding

> Note : Beware that actual cloud resources often have restrictions in allowed
> names. Some resources, for example, can't contain dashes, some must be
> camelCased. The conventions in this document refer to Terraform names
> themselves.


## Resource and/or data source naming

Avoid stuttering when naming Terraform resources. Examples : 

```terraform
resource "aws_route_table" "public" {}                 # <- 🟢 no stuttering 
resource "aws_route_table" "public_route_table" {}     # <- 🔴 stuttering
resource "aws_route_table" "public_aws_route_table" {} # <- 🔴 maximum stuttering

Name a resource `this` if there is no more descriptive and general name available, or if the resource is part of a module that creates a single resource of this type. For example, in a `terraform-google-bucket` module : 

```terraform
resource "google_storage_bucket" "this" {
  name = var.name
  // ...
}

Resource and module instance names should always be singular.

## Variables

Variable names for **lists** or **maps** must be plural.

Avoid repeating yourself within a variable's name. Example :
```terraform
variable "s3_storage_bucket_name" # <- 🔴 too precise
variable "bucket"                 # <- 🔴 not precise enough
variable "bucket_name"            # <- 🟢 seems like a good compromise
```

Always provide a variable's description, even if it seems obvious to you.

```terraform
variable "tags" {
  description = "A mapping of tags to assign to the resource."
  type        = map(string)
  default = {
    terraform = "true",
  }
}
```

Do not reinvent the wheel when naming variables or writing descriptions. If a
variable is consumed by a resource or module directly, reuse the variable
name from the "**Argument Reference**" section in the resource's (or module's)
documentation.

## Module outputs

Naming module outputs properly is important for them to be consistent through
your codebase, and for them to be understandable outside of their immediate
context. From a user's perspective, it should be trivial what type each
attribute has.

If the output is a **list** or a **map**, its name must be plural.

Always provide an output's description, even if it seems obvious to you.

Example :

```terraform
output "this" {
  description = "The AKS resource."
  value       = data.azurerm_kubernetes_cluster.this
}
```

Example of a module that instanciates a Function App, a Storage Account and an
App Insights. The module is called `azurerm-function-app`.

```terraform
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
