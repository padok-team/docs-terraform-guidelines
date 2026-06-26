# Terragrunt
## Context pattern <!-- omit in toc -->

<!-- markdownlint-disable MD051 -->
- [✒️ Definition](#️-definition)
- [🎯 Recommended project architecture](#-recommended-project-architecture)
- [📏 Rules to respect](#-rules-to-respect)
- [🗃️ Files and folder naming](#️-files-and-folder-naming)
  - [📁 Folders](#-folders)
  - [📄 Files](#-files)
    - [`root.hcl`](#roothcl)
    - [`module.hcl`](#modulehcl)
    - [`inputs.hcl`](#inputshcl)
    - [`terragrunt.hcl`](#terragrunthcl)
- [⚖️ Pros and cons](#️-pros-and-cons)

---

## ✒️ Definition

**Context pattern** : the **DRY** application of [WYSIWYG](../terraform/wysiwg_patterns.md) through Terragrunt. Each layer instantiates a **single module**, and its inputs (its "context") are declared **once** and inherited down the tree structure.

> [!NOTE]
> A design pattern is a general, reusable solution to a commonly occurring problem within a given context in software design

 **DRY**: *Don't Repeat Yourself* is the core motto of [Terragrunt](https://terragrunt.gruntwork.io/).

Terragrunt allows you to:

- Write each configuration **once** across your infrastructure
- Keep your **backend config** DRY
- Enforce **separation of duty** between your layers

---

## 🎯 Recommended project architecture

```txt
.
├── layers
│   ├── bastion
│   │   ├── module.hcl                 # The module called by every layer below
│   │   ├── dev
│   │   │   ├── inputs.hcl             # Inputs unique to this layer
│   │   │   └── terragrunt.hcl         # 👈 The layer to apply (deepest file)
│   │   └── prod
│   │       ├── inputs.hcl             # Inputs unique to this layer
│   │       └── terragrunt.hcl         # 👈 The layer to apply (deepest file)
│   └── application
│       ├── module.hcl                 # The module called by every layer below
│       ├── dev
│       │   ├── inputs.hcl             # Inputs unique to this layer
│       │   └── terragrunt.hcl         # 👈 The layer to apply (deepest file)
│       └── prod
│           ├── inputs.hcl             # Inputs unique to this layer
│           └── terragrunt.hcl         # 👈 The layer to apply (deepest file)
├── modules                            # Local Terraform modules (optional)
└── root.hcl                           # Root config shared by every layer
```

---

## 📏 Rules to respect

Five rules drive this pattern:

- 🟢 **`terragrunt.hcl` is always the deepest file of a branch.** It marks a
  *layer*: the directory where you run `terragrunt apply`. Nothing lives below
  it.
- 🟢 **No `inputs.hcl` below the layer to apply.** Inputs are defined *at* the
  layer or *above* it, never deeper than the `terragrunt.hcl` they feed.
- 🟢 **Only four configuration file names exist:** `root.hcl`, `module.hcl`,
  `inputs.hcl` and the `terragrunt.hcl` leaf that defines the layer.
- 🟢 **Context: your tree, with its inputs, is your context.** If your context is
  DRY, your tree structure is OK; if you have to repeat an input, you might want
  to refactor your tree structure (see
  [Refactoring with Terragrunt](../operations/refactoring.md#refactoring-with-terragrunt)).
- 🟢 **Uniqueness: each layer defines one and only one module.** You cannot
  create a new resource independently without adding it to the module, which
  forces you to ask the questions:

  > Should this new resource really be added to this layer?\
  > Isn't this resource a new project need?

  Consequently, you can either:

  - **Integrate** it with the current layer module, if this configuration will be generalized
  - **Create a new tree structure**, if a new project need has been identified

---

## 🗃️ Files and folder naming

### 📁 Folders

- **Root folders** are named `layers/` for terragrunt files and `modules/` for terraform files
- **Layer folders** are named after the business need they address; it is recommended to name them identically to the corresponding module folder
- **Module folders** are named after what they define

### 📄 Files


| File | Where | Role |
| --- | --- | --- |
| `root.hcl` | Repository root | Configuration shared by **every** layer: remote backend, generated provider/version block, global inputs. |
| `module.hcl` | Above the layers that share a module | Declares the **single** Terraform module the layers below it instantiate and the Terragrunt layers it optionally depends on. |
| `inputs.hcl` | At a layer or any parent | Inputs for that layer overriding or completing the values defined higher up. |
| `terragrunt.hcl` | The leaf = the layer to apply | The unit you `terragrunt apply`. Wires the includes together. |

#### `root.hcl`

Defines what **every layer shares**: the remote state backend (with a **unique key
per layer**) and a `generate` block for the Terraform `required_version` and
providers. Global inputs can also live here.

```hcl
# root.hcl
terraform_version_constraint  = "~> x.x.x"
terragrunt_version_constraint = "~> x.x.x"

locals {
  common_tags = {
    owner = "pitouna@email.fr"
  }
  environment = basename(get_original_terragrunt_dir())
}

remote_state {
  backend = "s3"
  generate = {
    path      = "backend.tf"
    if_exists = "overwrite"
  }
  config = {
    bucket = "my-project-tfstate"
    key    = "${path_relative_to_include()}/terraform.tfstate" # 👈 unique per layer
    region = "eu-west-3"
  }
}

generate "provider" {
  path      = "_settings.tf" # generated file, prefixed with _
  if_exists = "overwrite"
  contents  = <<EOF
terraform {
  required_version = "= 1.6.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "= 5.0.0"
    }
  }
}
EOF
}
```

#### `module.hcl`

Declares the **single module** that the layers below it instantiate. **Pin remote
modules to a tag.**
You can use outputs from others layers with **dependencies**.

```hcl
# layers/bastion/module.hcl
terraform {
  source = "${get_repo_root()}//modules/bastion"
}

locals {
  root = read_terragrunt_config(find_in_parent_folders("root.hcl"))
}

# optional : only needed when this module consumes outputs from another layer
dependency "vpc" {
  config_path = "../../vpc/${local.root.locals.environment}"
}

inputs = {
  context    = local.root.locals.common_tags
  subnet_ids = dependency.vpc.outputs.private_subnet_ids # output exposed by the vpc layer
}

```

#### `inputs.hcl`

Holds the inputs **specific** to a layer (or a shared parent). Anything common
to several layers should **move up the tree** to stay DRY.

```hcl
# layers/database/dev/inputs.hcl
inputs = {
  database_size  = "db.t3.micro" # smaller than production
  backup_enabled = false
}
```

#### `terragrunt.hcl`

The **leaf**, and the only file you apply. It wires together the root config, the
parent `module.hcl`, and the local `inputs.hcl`.

```hcl
# layers/environment/dev/terragrunt.hcl
include "root" {
  path           = find_in_parent_folders("root.hcl")
  merge_strategy = "deep"
}

include "module" {
  path           = find_in_parent_folders("module.hcl")
  merge_strategy = "deep"
}

include "inputs" {
  path           = "inputs.hcl"
  merge_strategy = "deep"
}

```

---

## ⚖️ Pros and cons

**Pros:**

- Very easy to create a new layer (project need)
- Feature segmentation with modules
- Easy to navigate: what you see is what you get (**WYSIWYG**)
- **DRY** 🌞
- Enforces best practices when creating a new resource
- Easy to refactor (one layer = one module = one state, see
  [Refactoring with Terragrunt](../operations/refactoring.md#refactoring-with-terragrunt))

**Cons:**

- Adds a tool to your **stack**: an extra dependency, a learning curve, and one
  more layer of indirection to debug.
- The **DRY trade-off** works against pure WYSIWYG: a layer's full set of inputs is
  spread across the tree (`root.hcl`, parent `inputs.hcl`, `module.hcl`), so you
  have to follow the `include` chain to know exactly what a layer applies.
- **Generated files** (`_settings.tf`, `backend.tf`) are not the source of truth:
  what you edit and what Terraform sees differ.
