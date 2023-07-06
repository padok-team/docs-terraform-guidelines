# Creating a good IaC structure

- [Creating a good IaC structure](#creating-a-good-iac-structure)
  - [Objective](#objective)
  - [Flow](#flow)
    - [Are you creating an infrastructure from scratch ?](#are-you-creating-an-infrastructure-from-scratch-)
      - [1 team (The operational team)](#1-team-the-operational-team)
        - [1 environment or multienvironment resources](#1-environment-or-multienvironment-resources)
        - [2 or + environments](#2-or--environments)
          - [Applications (default approach)](#applications-default-approach)
          - [Environment](#environment)
      - [2 or more teams](#2-or-more-teams)
    - [Are you contributing to an existing code base ?](#are-you-contributing-to-an-existing-code-base-)
      - [Have you identified where you will add your resources ?](#have-you-identified-where-you-will-add-your-resources-)

## Objective

This documentation aims to help you choose the best code structure for your project

## Flow

### Are you creating an infrastructure from scratch ?
  
Great ! First, how many teams will work on this infrastructure ?

#### 1 team (The operational team)

Only the operational team will be creating components in this infrastructure. Make sure that you know how will devs (and possibly data engineer) deploy their code and who creates what they will need to host them.

How much segmentation do you need to create your resources ? (Sandbox, dev, staging, production or is it global to all environments)

##### 1 environment or multienvironment resources

You could only need one if you're a creating a share network between environment, it's best to code it in the same place. This is very useful if you are building the hub of a hub and spoke.
It can also be used to manage IAM users or rules.

➡️ Divide your repository by project need (Network, DNS, IAM)

➡️ Don't hesitate to create sub layer for complex project needs. Per example, have different network configuration in and out of production.

```tree
.
├── layers
│   ├── gateways
│   ├── dns-and-private-ca
│   ├── github-runners
│   ├── load-balancers
│   │   ├── non-production
│   │   └── production
│   ├── subnet-routing
│   │   ├── dev
│   │   ├── network
│   │   ├── production
│   │   ├── sandbox
│   │   ├── staging
│   │   └── test
│   ├── transit-gateway-routing
│   │   ├── hub
│   │   └── spoke
│   ├── vpc
│   │   ├── dev
│   │   ├── network
│   │   ├── production
│   │   ├── sandbox
│   │   ├── staging
│   │   └── test
│   ├── vpc-endpoint
│   └── vpn
└── modules
    ├── dns-and-private-ca
    ├── gateways
    ├── github-runners
    ├── load-balancers
    ├── subnet-routing
    ├── transit-gateway-routing
    ├── vpc
    ├── vpc-endpoint
    └── vpn
```

This code base has been used on:

- An airline company

##### 2 or + environments

This is the most classic setup, having multiple environment to have segmentation between stages of release.

To organize your code, you will need to reflect on what you are most likely to create regularly ?

- Do you create a new application regularly, because you have a microservice approach to your architecture ?
  - Go for the `Applications` approach.
- Do you have a more centralized approach to your application ?
  - Go for the `Environment` approach.

###### Applications (default approach)

This approach gives more responsibility on each application

➡️ IaC application oriented

```tree
.
├── layers
│   ├── application-1
│   │   ├── app.hcl
│   │   └── fr
│   │       ├── dev
│   │       │   ├── app-type-2
│   │       │   │   └── terragrunt.hcl
│   │       │   ├── database
│   │       │   │   └── terragrunt.hcl
│   │       │   ├── ecs
│   │       │   │   └── terragrunt.hcl
│   │       │   └── env.hcl
│   │       └── region.hcl
│   └── application-2
│       ├── app.hcl
│       └── fr
│           ├── dev
│           │   ├── app-type-1
│           │   │   └── terragrunt.hcl
│           │   └── env.hcl
│           └── region.hcl
└── modules
```

###### Environment  

This approach gives more responsibility on the platform team. It's a good approach if you have a lot of applications and you want to have a more centralized approach.

➡️ IaC environment oriented

```tree
.
├── layers
│   └── fr
│       ├── dev
│       │   ├── application-1
│       │   │   ├── app.hcl
│       │   │   ├── bucket
│       │   │   │   └── terragrunt.hcl
│       │   │   ├── database
│       │   │   │   └── terragrunt.hcl
│       │   │   └── ecs
│       │   │       └── terragrunt.hcl
│       │   ├── application-2
│       │   │   ├── app.hcl
│       │   │   └── bucket
│       │   │       └── terragrunt.hcl
│       │   └── env.hcl
│       └── region.hcl
└── modules
```

#### 2 or more teams  

You can have multiple operational teams. One for the network, IAM or even data engineers.

➡️ Create a repository for each team to separate git flows

➡️ You can then follow the guidelines for `1 team` for each team.

### Are you contributing to an existing code base ?
  
TODO

#### Have you identified where you will add your resources ?  
