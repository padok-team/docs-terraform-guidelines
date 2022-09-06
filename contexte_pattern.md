# Context pattern <!-- omit in toc -->

- [Contexte pattern](#contexte-pattern)
  - [Recommended project architecture](#recommended-project-architecture)
  - [Context pattern](#context-pattern)
  - [Terragrunt context proposal](#terragrunt-context-proposal)
    - [Input heaven](#input-heaven)
    - [Context](#context)
    - [Uniqueness](#uniqueness)
    - [Ease of refactoring](#ease-of-refactoring)
      - [Example 1 : Add a one frontend](#example-1--add-a-one-frontend)
      - [Example 2 : Add a redis to all backends](#example-2--add-a-redis-to-all-backends)
  - [Files and folder naming](#files-and-folder-naming)
    - [Folders](#folders)
    - [Files](#files)
    - [⚖️ Pros and cons](#️-pros-and-cons)

> A design pattern is a general, reusable solution to a commonly occurring problem within a given context in software design

## Recommended project architecture

```txt
.
├── environment 
│   ├── module.hcl
│   ├── dev  
│   │   ├── inputs.hcl 
│   │   └── terragrunt.hcl 
│   ├── preproduction
│   │   ├── inputs.hcl 
│   │   └── terragrunt.hcl
│   └── production
│       ├── inputs.hcl 
│       ├── organisation.tf
│       └── terragrunt.hcl 
├── application 
│   ├── app_1  
│   │   ├── module.hcl
|   │   ├── dev  
|   │   │   ├── inputs.hcl 
|   │   │   └── terragrunt.hcl
|   │   ├── preproduction  
|   │   │   ├── inputs.hcl 
|   │   │   └── terragrunt.hcl
|   │   └── production  
|   │      ├── inputs.hcl 
|   │      └── terragrunt.hcl
│   ├── app_2
│   └── app_3
├── common.hcl
└── _settings.hcl
```

## Context pattern

You implemented the [WSIWYG pattern](wysiwg_patterns.md) and have copied module between your layers and are tired of copying them or made some mistakes while doing so.

The context pattern for **WYSIWYG** aims to keep your code DRY (Don't repeat yourself)! 🌞

Thus all concepts from the **WYSIWYG** pattern apply to this pattern.

## Terragrunt context proposal

Don't repeat yourself is the motto of terragrunt.
Terragrunt allows you:

- To write your configuration through out your infrastructure
- To keep your backend config dry
- Enforce separation of duty between your layers

### Input heaven

Each terragrunt layer defines one and only one module, but input for this module can be defined through out the tree structure.
Example:

```txt
.
├── environment 
│   ├── module.hcl # Defines the module used by every subsequent layer
│   └─── dev  
│       ├── inputs.hcl # Defines **unique** input for this layer (Ex: the database should be a bit smaller in dev)
│       └── terragrunt.hcl # Defines terragrunt configuration
├── common.hcl # Defines default value to every layer (Ex: tenant id, size of the database)
└── _settings.hcl # Defines required_version of terraform, terraform provider used but not the version and unique identifier for the backend **for every layer**
```

### Context

The context of a layer is all the inputs required for it to be created.
Your tree structure with inputs through out it, is thus your context.

If your context is DRY then your tree structure is OK, if you have to repear an input you might want to refactor your tree structure. Check [refactoring section](#ease-of-refactoring)
if you are dry your arbo is OK if not you should change to simplify

### Uniqueness

Each terragrunt layer defines one and only one module, you can not create a new resource independently without adding it to the module.
This will force you to ask the question

> should this new resource really be added to this layer ?
> Isn't this resource a new project need ?

Consequently you can either:

- Integrate it with the current layer module because this configuration will be generalized
- Create a new tree structure because a new project need has been identified

### Ease of refactoring

The bigest advantage of terragrunt is that since every layer is a single module thus a single state. When identifying a new project need you can rearrange you tree structure to match the new view of your project need.

#### Example 1 : Add a one frontend

```txt
.
├── application 
│   ├── module.hcl # Calls the backend module
│   └─── app_1
|         ├── dev
│         │    ├── inputs.hcl 
│         │    └── terragrunt.hcl 
│         └── production
│              ├── inputs.hcl 
│              └── terragrunt.hcl
└── _settings.hcl
```

> My application needs a frontend but only for this one app

```txt
.
├── application 
│   ├── backend
│   │   ├── module.hcl
│   │   └─── app_1  
│   │         ├── dev
│   │         │   ├── inputs.hcl 
│   │         │   └── terragrunt.hcl 
│   │         └── production
│   │             ├── inputs.hcl 
│   │             └── terragrunt.hcl
│   └── frontend
│       ├── module.hcl
│       └─── app_1  
│             ├── dev
│             │   ├── inputs.hcl 
│             │   └── terragrunt.hcl 
│             └── production
│                 ├── inputs.hcl 
│                 └── terragrunt.hcl
└── _settings.hcl
```

#### Example 2 : Add a redis to all backends

```txt
.
├── application 
│   ├── module.hcl # Calls the backend module
│   └─── app_1
|         ├── dev
│         │    ├── inputs.hcl 
│         │    └── terragrunt.hcl 
│         └── production
│              ├── inputs.hcl 
│              └── terragrunt.hcl
└── _settings.hcl
```

> I'm going to need a redis for my backend
> I'll juste update my module backend and test it by overwriting the call to the module within the layer terragrunt.hcl

## Files and folder naming

### Folders

- Root folder are named depending on project need (Ex: application, environment)
- Layer folder are named depending on subproject need (Ex: production, dev, app_1)
- Module folder are named after what they define

### Files

- The file `_settings.hcl` defines required_version of terraform, terraform provider used but not the version and unique identifier for the backend
- The file `input.hcl` defines all the inputs spcefique to the layer
- The file `module.hcl` defines the module used by subsequent layers
- The file `terragrunt.hcl` defines in every layer the terragrunt configuration of include
- The file `common.hcl` defines default value to every layer

### ⚖️ Pros and cons

Pros:

- Very easy to create a new layer (project need)
- Segmentation of feature with module
- Ease of exploration : What you see is what you get (WYSIWYG)
- DRY 🌞
- Enforce best practices when creating a new resource
- Ease of refactoring
