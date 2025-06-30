# Optional attributes <!-- omit in toc -->

## Update

This guideline was written while this feature was still an experiment in Terraform. As of [Terraform 1.3.0](https://github.com/hashicorp/terraform/blob/v1.3/CHANGELOG.md#130-september-21-2022), it's generally available and we use it internally without encountering issues.

Therefore, we now recommend the use of this feature while considering the risks that we outlined below. We also created a new section [How you can safely use this feature](#how-you-can-safely-use-this-feature) to help you use it safely.

## Introduction

[Optional attributes](https://developer.hashicorp.com/terraform/language/expressions/type-constraints#optional-object-type-attributes) allow Terraform to insert a default value when an object attribute is undefined.

- [Update](#update)
- [Introduction](#introduction)
- [How you can safely use this feature](#how-you-can-safely-use-this-feature)
  - [Specify the default value](#specify-the-default-value)
  - [Be aware of typos](#be-aware-of-typos)
- [How you could solve your problem differently](#how-you-could-solve-your-problem-differently)
  - [Option 1: delegate the defaulting logic to the caller](#option-1-delegate-the-defaulting-logic-to-the-caller)
  - [Option 2: rework your module's interface](#option-2-rework-your-modules-interface)

## How you can safely use this feature

### Specify the default value

It's possible to specify the default value of an optional attribute, like in this example :

```hcl
variable "with_optional_attribute" {
  type = object({
    a = string                # a required attribute
    b = optional(string)      # an optional attribute 🔴
    c = optional(number, 127) # an optional attribute with a default value 🟢
  })
}
```

You should always specify the default value of an optional attribute. This way, you won't need to handle the `null` value for your attribute in your module's implementation, and therefore avoid runtime errors if you forget to handle it.

### Be aware of typos

When using this feature, Terraform will not warn you if you make a typo in the name of an attribute. For example, if you write `procotol` instead of `protocol`, Terraform will not complain and will use the default value for `protocol`.

Below this section is the former content of this guideline. We keep it to explain why we changed our mind about this feature.

## How you could solve your problem differently

The problem you are trying to solve is one of an over-complicated interface.
This is a very legitimate problem.

### Option 1: delegate the defaulting logic to the caller

If your module does not implement any defaulting logic, the caller of your
module can still implement it:

```terraform
locals {
    endpoints = [
        {
            path    = "/api/v1"
            backend = "http://legacy-api:8080/"
        },
        {
            path    = "/api/v2"
            backend = "http://new-api:8080/"
        },
        {
            protocol      = "FTP"
            path          = "/"
            backend       = "ftp://ftp-server/"
            ssl_enabled   = false
            auth_required = false # server handles auth itself
        },
    ]
}
module "gateway" {
    source = "..."
    endpoints = [
        for endpoint in local.endpoints:
        {
            protocol      = lookup(endpoint, "protocol", "HTTP")
            path          = endpoint.path
            backend       = endpoint.backend
            ssl_enabled   = lookup(endpoint, "ssl_enabled", true)
            ssl_redirect  = lookup(endpoint, "ssl_redirect", true)
            auth_required = lookup(endpoint, "auth_required", true)
        }
    ]
}
```

You could even include an example of this logic in your module's documentation.

This option is not perfect: it will not catch typos in the `local.endpoints`
variable.

### Option 2: rework your module's interface

The best modules have rich implementations and simple interfaces. If you feel
the need to simplify your module's interface, then you know that it is not
simple.

It may be time to think about refactoring. Use the understanding you have of the
problem you are trying to solve to create a better interface. This interface
will not necessarily do the same things as your current one. Maybe it will
include more than one module, or have fewer features, there are many
possibilities.

Try to take a step back and imagine what you want the calling code to look like,
and what complexity you want to remove from the calling code. This is a great
place to start when designing a new module.

[optional-attributes]: https://www.terraform.io/language/expressions/type-constraints#experimental-optional-object-type-attributes
