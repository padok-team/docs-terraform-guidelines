# Iterate on your resources

In your Terraform code, you may need to declare a resource or a module multiple times, and you don't want to repeat yourself. In order to do so, there are multiple ways.

## TL;DR

> Prefer using a `for_each` than a `count` when you want to declare a resource multiple times with some changes
> Use `count` mechanism for feature flags when you want to activate or not a part of your code

## Object iteration (`for_each`)

When you want to declare the same resource type multiple times, with some configuration changes in each occurence (take an azurerm_network_security_rule in an azurerm_network_security_group for example), you will prefer to use a map of objects and iterate on it to configure your resources.

Using this feature allow you to attribute a named key to your declaration and to clearly identify your resources in your terraform code. When you will want to delete an instance, terraform will be able to find it by its key and delete it without any impact on other instance in the map.

**terraform code using `for_each`:**

```json=

locals {
    nsr_map = {
        "rule-1" = {
            source_address_prefix = "10.1.0.0/28"
        },
        "rule-2" = {
            source_address_prefix = "10.2.1.0/28"
        },
    }
}

resource "azurerm_network_security_group" "example" {
    name = "example"
    ...
}

resource "azurerm_network_security_rule" "examples" {
    for_each = local.nsr_map
    name = each.key
    source_address_prefix = each.value.source_address_prefix
}
```

**state list of a `for_each`:**

```bash=
terraform state list

azurerm_network_security_group.example
azurerm_network_security_rule.examples["rule-1"]
azurerm_network_security_rule.examples["rule-2"]
```

However, it's not recommended to use a `for_each` by creating a map from a list, because lists are deterministic and it will enforce the recreation of each instance. Besides, think about changing your list into another type.

**bad use of `for_each`:**

```json=
locals {
    nsr_list = [
        { "name": "rule-1", source_address_prefix = "10.0.1.0/28" },
        { "name": "rule-2", source_address_prefix = "10.0.2.0/28" },
    ]
}

resource "azurerm_network_security_group" "example" {
    name = "example"
    ...
}

resource "azurerm_network_security_rule" "examples" {
    for_each = {for v in local.nsr_list : v.name => v.source_address_list }
    name = each.key
    source_address_prefix = each.value
}
```

## List iteration (`count`)

Iterate on a list using a `count` feature is not a good solution for many reasons. Most of the time, you will want to create resources with some changes in each iteration and you will need to iterate on a third-part list of objects in order to pass configurations. Your terraform code will be more complex to understand with to much logic in it.

Moreover, using count feature enforce resources deletion / recreation when you remove an item in the middle of your iteration list. If your terraform code has some design issues, you may destroy resources you don't want to.

**terraform code using `count`:**

```json=
locals {
    nsr_list = [
        { "name": "rule-1", source_address_prefix = "10.0.1.0/28" },
        { "name": "rule-2", source_address_prefix = "10.0.2.0/28" },
    ]
}

resource "azurerm_network_security_group" "example" {
    name = "example"
    ...
}

resource "azurerm_network_security_rule" "examples" {
    count = length(local.nsr_list)
    name = local[count.index].name
    source_address_prefix = local[count.index].source_address_prefix
}
```

Nevertheless, `count` feature is a good way to create `if` like behaviour in terraform. You may need it create some `feature flags` in your terraform code.

**feature flag code using `count`:**

```json=
locals {
    enable_nsr = false
}

resource "azurerm_network_security_group" "example" {
    name = "example"
    ...
}

resource "azurerm_network_security_rule" "example" {
    count = local.enable_nsr ? 1 : 0
    name = "rule-1"
    source_address_prefix = "10.0.0.1/28"
}
```
