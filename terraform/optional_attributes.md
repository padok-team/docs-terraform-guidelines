# Optional attributes <!-- omit in toc -->

## Update

This guideline was written while this feature was still an experiment in Terraform. As of [Terraform 1.3.0](https://github.com/hashicorp/terraform/blob/v1.3/CHANGELOG.md#130-september-21-2022), it's generally available and we use it internally without encountering issues.

Therefore, we now recommend the use of this feature while considering the risks that we outlined below. We also created a new section [How you can safely use this feature](#how-you-can-safely-use-this-feature) to help you use it safely.

## Introduction

[Optional attributes][optional-attributes] are an experimental feature that you
can enable in a Terraform module. When enabled, the module's implementation can
make use of the `optional` and `defaults` functions.

- [Update](#update)
- [Introduction](#introduction)
- [How you can safely use this feature](#how-you-can-safely-use-this-feature)
  - [Specify the default value](#specify-the-default-value)
  - [Be aware of typos](#be-aware-of-typos)
- [Executive summary](#executive-summary)
- [Why you may want to use this feature](#why-you-may-want-to-use-this-feature)
- [Why you should not use this feature](#why-you-should-not-use-this-feature)
  - [Reason 1: it can hide mistakes in the caller's code](#reason-1-it-can-hide-mistakes-in-the-callers-code)
  - [Reason 2: it can scare users of your module](#reason-2-it-can-scare-users-of-your-module)
  - [Reason 3: it can stop you from upgrading Terraform](#reason-3-it-can-stop-you-from-upgrading-terraform)
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
    b = optional(string)      # an optional attribute
    c = optional(number, 127) # an optional attribute with a default value
  })
}
```

You should always specify the default value of an optional attribute. This way, you won't need to handle the `null` value for your attribute in your module's implementation, and therefore avoid runtime errors if you forget to handle it.

### Be aware of typos

When using this feature, Terraform will not warn you if you make a typo in the name of an attribute. For example, if you write `procotol` instead of `protocol`, Terraform will not complain and will use the default value for `protocol`.

Below this section is the former content of this guideline. We keep it to explain why we changed our mind about this feature.

## Executive summary

Do not use this feature. It is unstable and makes your module easier to misuse.

In general, do not force users of a module to use experimental features.

This feature is a code smell: the module's interface is most likely too complex.

## Why you may want to use this feature

Long story short, because your module expects objects as variables and some of
the attributes have default values.

Your module may have a complex variable with many fields, like this:

```terraform
variable "endpoints" {
    type = list(object({
        protocol     = string
        path         = string
        backend      = string
        ssl_enabled  = bool
        ssl_redirect = bool
        auth_enabled = bool
    }))
}
```

And you call your module this way:

```terraform
module "gateway" {
    source = "..."

    endpoints = [
        {
            protocol      = "HTTP"
            path          = "/api/v1"
            backend       = "http://legacy-api:8080/"
            ssl_enabled   = true
            ssl_redirect  = true
            auth_required = true
        },
        {
            protocol      = "HTTP"
            path          = "/api/v2"
            backend       = "http://new-api:8080/"
            ssl_enabled   = true
            ssl_redirect  = true
            auth_required = true
        },
        {
            protocol      = "FTP"
            path          = "/"
            backend       = "ftp://ftp-server/"
            ssl_enabled   = false
            ssl_redirect  = false # no SSL so no redirect required
            auth_required = false # server handles auth itself
        },
    ]
}
```

Calling that module is repetitive because most values don't change that often.
It would be nicer for the caller to write:

```terraform
module "gateway" {
    source = "..."

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
```

The `protocol` defaults to `HTTP`, which is most common. SSL is enabled by
default, with redirection. If SSL is disabled, so is SSL redirection.
JWT authentication is required unless specified otherwise.


## Why you should not use this feature

### Reason 1: it can hide mistakes in the caller's code

A user of your module may make a typo, like this (can you spot it?):

```terraform
module "gateway" {
    source = "..."

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
            procotol      = "FTP"
            path          = "/"
            backend       = "ftp://ftp-server/"
            ssl_enabled   = false
            auth_required = false # server handles auth itself
        },
    ]
}
```

Terraform ignores the unknown `procotol` attribute and, since the experimenntal
feature is enabled, defaults `protocol` to `HTTP`. There is no syntax error in
the caller's code, nothing for `terraform validate` to catch. The user of your
module may spend hours figuring out why their FTP server is not working as
expected.

### Reason 2: it can scare users of your module

When applying code that calls your module with a Terraform version priori to
1.3.0, Terraform prints a large warning if the feature is enabled. This is
intimidating for some of Padok's clients, who wonder why their brand new
codebase already has warnings about potential instability.

Actually, in Terraform versions prior to 1.3.0, the official documentation read:

> Until the experiment is concluded, the behavior of this feature may see
> breaking changes even in minor releases. We recommend using this feature only
> in prerelease versions of modules as long as it remains experimental.

Even when using Terraform versions following 1.3.0, **Reason 1** still applies,
so this feature should still be avoided.

### Reason 3: it can stop you from upgrading Terraform

Modules that enable the experiments = [module_variable_optional_attrs] in
Terraform settings will not work with Terraform versions greater than 1.3.0.
Actually, you will get an error when running `terraform init`. Therefore, using
this feature makes it harder to upgrade from Terraform < 1.3 to Terraform
>= 1.3, since it requires updating your codebase as well.

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
