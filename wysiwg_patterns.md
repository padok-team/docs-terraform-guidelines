# Terraform design pattern WYSIWYG (What you see is what you get)

- [Terraform design pattern WYSIWYG (What you see is what you get)](#terraform-design-pattern-wysiwyg-what-you-see-is-what-you-get)
  - [🏗 Recommended project architecture](#-recommended-project-architecture)
  - [🦮 Guidelines](#-guidelines)
    - [Layers, folders and projects](#layers-folders-and-projects)
      - [Size of layer](#size-of-layer)
    - [Modules](#modules)
    - [Versioning](#versioning)
  - [Files and folder naming](#files-and-folder-naming)
    - [Folders](#folders)
    - [Files](#files)
  - [⚖️ Pros and cons](#️-pros-and-cons)
  - [Examples](#examples)
    - [Example 1 : Project with a seperation per environment, module are in local repository with no versioning](#example-1--project-with-a-seperation-per-environment-module-are-in-local-repository-with-no-versioning)
    - [Example 2 : Project with a seperation per application, module are in remote repository with versioning](#example-2--project-with-a-seperation-per-application-module-are-in-remote-repository-with-versioning)

> Design pattern is a general, reusable solution to a commonly occurring problem within a given context in software design

## 🏗 Recommended project architecture

```txt
.
├── environment 
│   ├── dev  
│   │   ├── _settings.tf  
│   │   ├── output.tf 
│   │   ├── database.tf # Calls a local database module
│   │   └── environment.tf # Calls a local environment module
│   ├── preproduction
│   │   ├── _settings.tf  
│   │   ├── output.tf 
│   │   └── environment.tf # Calls a local environment module
│   └── production
│       ├── _settings.tf  
│       ├── output.tf 
│       ├── organisation.tf # Calls aresource directly
│       └── environment.tf  # Calls a local environment module
├── application 
│   ├── app_1  
|   │   ├── dev  
|   │   │   ├── _settings.tf  
|   │   │   ├── output.tf 
|   │   │   ├── rds.tf 
|   │   │   └── app.tf # Calls a remote app module
|   │   ├── preproduction  
|   │   │   ├── _settings.tf  
|   │   │   ├── output.tf 
|   │   │   └── app.tf # Calls a remote app module
|   │   └── production  
|   │      ├── _settings.tf  
|   │      ├── output.tf 
|   │      └── app.tf # Calls a remote app module
│   ├── app_2
│   └── app_3
└── modules
    ├── environment
    │   ├── README.md
    │   ├── _settings.tf
    │   ├── bastion.tf
    │   ├── bucket.tf
    │   ├── dns.tf
    │   ├── ip_ranges.tf 
    │   ├── eks.tf
    │   └── variables.tf 
    └── database
```

## 🦮 Guidelines

> Keep it simple stupid.

**Don’t put too much thought on the friendly name of files, the point is that a new comer can easily find code to understand the infrastructure and make modification.**

### Layers, folders and projects

A layer is a directory in which you use `apply/destroy` resource on your provider, thus a terraform state.

A folder is used to regroup these layers, and it should be named with a project need in mind (Ex: environments, applications).

A project can host multiple folder to regroup different type of project need.

- This folder contains layer that answer project needs (Ex: having a production, pre-production and tooling environment)
  - Each file in a layer is named with a friendly name (Ex: production, cluster, database, roles)
  - Can be seperate with sub folder to have sub layers, thus a state per sub layer and not layer. (Ex: If you state is too big with too many resource you can divide it in different sub layer)
  - Only one file for the configuration of terraform for the layer named `_settings.tf`
    - Defines required_version of terraform
    - Terraform provider used but not it’s version
    - Backend configuration

#### Size of layer

The size of a layer is important when deciding the segmentation of folder and layers within a project.

The following factors are indication of too large states

- The time it takes for a terraform plan/apply to refresh the state
- The number of collaborators
  - The more collaborators there are the more state locking incident will happen)
- Ownership of different team
  - A layer should only be owned by one team

### Modules

Modules are created for two reasons:

- If you need to create a resource multiple times (Ex: network, databases) ✅
- If you want to hide complexity to create a resource (Ex: EKS cluster) ✅
- They are **not** created to pass-through another module (Ex for public module) ❌

Each file in a module is named with a friendly name (Ex: node_pool, roles, monitoring)

Module can be be located either

- In the local repository
- In a remote repository

### Versioning

Versioning of IAC is important to assure the resilience of an infrastructure. It allows you to:

- Rest a breaking change on a module without breaking other `layers` using the same module
- Rollback easily in case of failure

This versioning can be done on several layers:

- Version the whole repo with the local modules
  - Pro :
    - Simple usage and quick implementation of new feature
    - Best suited for a one or two man team, synchronous collaboration
  - Cons
    - Not suited for teams of more than two persons and asynchronous collaboration
    - Need to copy module in a new folder `v2` and change source of module to test a breaking change
- The repo and the module are in remote repository
  - Pro :
    - Best suited for medium and large teams and asynchronous collaboration
    - Fine grain control on versioning
  - Con : Lead time to add a feature is long (Mulitple pull request to open)

## Files and folder naming

> Keep it simple stupid.

### Folders

- Root folder are named depending on project need (Ex: application, environment)
- Layer folder are named depending on subproject need (Ex: production, dev, app_1)
- Module folder are named after what they define

### Files

- The file `_settings.tf` Defines
  - Required_version of terraform
  - Backend configuration
  - Terraform provider used, but not it’s version.
- Terraform ouput of my layer ou module is named `output.tf` is the interface of my layer with the rest of the world
- Terraform file that create resources are named with a `reader_friendly_name.tf` after what they define
  - Do not use main.tf if possible

## ⚖️ Pros and cons

Pros:

- Very easy to create a new layer (project need)
  - business need
  - Security need
- Segmentation of feature with module
- Ease of exploration : What you see is what you get (WYSIWYG)

Cons

- Your layers are not always ISO between project need, because you can add .tf files to specify a ponctual need within a layer.
- With terraform you will have a bit of code duplication between layers, because they will each call the same module. These means you will have to write multiple times the same input for every instanciation of the module (Ex: Tenant id)
  - To solve this issue I invite you to read the [Contexte pattern](contexte_pattern.md)

## Examples

### Example 1 : Project with a seperation per environment, module are in local repository with no versioning

Main repository

```txt
.
├── environment 
│   ├── dev
│   │     ├── layer-1
│   │     │   ├── _settings.tf  
│   │     │   ├── output.tf 
│   │     │   ├── database.tf # Call local database module
│   │     │   └── environment.tf # Call local environment module
│   │     └── layer-2
│   │         ├── _settings.tf  
│   │         ├── output.tf 
│   │         └── kubernetes.tf # Call local kubernetes module
│   ├── preproduction
│   └── production
└── modules
    ├── environment
    │   ├── README.md
    │   ├── _settings.tf
    │   ├── bastion.tf
    │   ├── bucket.tf
    │   ├── dns.tf
    │   ├── ip_ranges.tf 
    │   ├── eks.tf
    │   └── variables.tf 
    └── database
```

### Example 2 : Project with a seperation per application, module are in remote repository with versioning

Main repository

```txt
.
└── apps 
    ├── api_1  
    |   ├── dev
    │   |  ├── _settings.tf  
    │   |  ├── output.tf 
    │   |  └── app.tf # Call remote module application
    |   ├── preproduction
    │   |  ├── _settings.tf  
    │   |  ├── output.tf 
    │   |  └── app.tf # Call remote module application
    |   └── production
    │      ├── _settings.tf  
    │      ├── output.tf 
    │      ├── datadog.tf # Call community module for Datadog
    │      └── app.tf # Call remote module application
    ├── api_2
    └── api_3
```

Remote module

```txt
.
└── application 
   ├── README.md
   ├── _settings.tf
   ├── bucket.tf
   ├── cloudrun.tf
   └── variables.tf 
```
