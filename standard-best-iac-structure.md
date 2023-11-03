# Creating a good IaC structure

- [Creating a good IaC structure](#creating-a-good-iac-structure)
  - [Objective](#objective)
  - [Are you creating an infrastructure from scratch ?](#are-you-creating-an-infrastructure-from-scratch-)
    - [1 team (The operational team)](#1-team-the-operational-team)
      - [1 environment or multienvironment resources](#1-environment-or-multienvironment-resources)
      - [2 or + environments](#2-or--environments)
        - [Applications (default approach)](#applications-default-approach)
          - [Application oriented - simple](#application-oriented---simple)
        - [Environments](#environments)
          - [Environment oriented - simple](#environment-oriented---simple)
          - [Environment oriented - complex](#environment-oriented---complex)
    - [2 or more teams](#2-or-more-teams)

## Objective

This documentation aims to help you choose the best code structure for your project

## Are you creating an infrastructure from scratch ?
  
Great ! First, how many teams will work on this infrastructure ?

### 1 team (The operational team)

Only the operational team will be creating components in this infrastructure. Make sure that you know how devs will (and possibly data engineer) deploy their code. Take also in consideration how they host their applications.

How much segmentation do you need to create your resources ? (Sandbox, dev, staging, production or is it global to all environments)

#### 1 environment or multienvironment resources

You could only need one if you're a creating a share network between environment, it's best to code it in the same place. This is very useful if you are building the hub of a hub and spoke.
It can also be used to manage IAM users or rules.

➡️ Divide your repository by project need (Network, DNS, IAM, ...)

➡️ Don't hesitate to create sub layer for complex project needs. Per example, have different network configuration in and out of production.

```tree
.
├── root.hcl
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

#### 2 or + environments

This is the most classic setup, having multiple environment to have segmentation between stages of release.

To organize your code, you will need to reflect on what you are most likely to create regularly ?

- Do you create a new application regularly, because you have a microservice approach to your architecture ?
  - Go for the `Applications` approach.
- Do you have a more centralized approach to your application ?
  - Go for the `Environment` approach.

##### Applications (default approach)

This approach gives more responsibility to each application team. It's a good approach if you have a lot of different applications, and you want to have a more decentralized approach.

###### Application oriented - simple

```tree
.
├── root.hcl
├── layers
│   ├── application-1
│   │   ├── app.hcl
│   │   └── fr
│   │       ├── dev
│   │       │   └── terragrunt.hcl
│   │       │   └── env.hcl
│   │       └── region.hcl
│   └── application-2
│       ├── app.hcl
│       └── fr
│           ├── dev
│           │   └── terragrunt.hcl
│           │   └── env.hcl
│           ├── test
│           │   └── terragrunt.hcl
│           │   └── env.hcl
│           └── region.hcl
└── modules
```

This code base has been used on:

- A business to retail company that helps client pay their restaurant bill.

##### Environments

This approach gives more responsibility on the platform team. It's a good approach if you have a lot of similar applications, and you want to have a more centralized approach.

For SaaS company, it's a good approach to have a tenant per client. The applications don't defer enough between deployment, but you want to manage dependencies centrally.
You will have a lot more layers in `production` than in any other environment because your clients won't necessarily have beta or staging environment.

PS: This approach has it limit. If you have a lot of clients, you will have a lot of layers, which can become hard to manage. To avoid having a giga-layer, you should look into other model to deploy your tenants. (ex: k8s controller)

###### Environment oriented - simple

```tree
.
├── root.hcl
├── layers
│   └── fr
│       ├── prod
│       │   ├── client-1
│       │   │       └── terragrunt.hcl
│       │   ├── client-2
│       │   │       └── terragrunt.hcl
│       │   └── env.hcl
│       ├── staging
│       │   ├── client-1 
│       │   │       └── terragrunt.hcl
│       │   └── env.hcl
│       └── region.hcl
└── modules
```

###### Environment oriented - complex

```tree
.
├── root.hcl
├── layers
│   └── fr
│       ├── prod
│       │   ├── client-1
│       │   │   ├── app.hcl
│       │   │   ├── bucket
│       │   │   │   └── terragrunt.hcl
│       │   │   ├── database
│       │   │   │   └── terragrunt.hcl
│       │   │   └── ecs
│       │   │       └── terragrunt.hcl
│       │   ├── client-2
│       │   │   ├── app.hcl
│       │   │   └── bucket
│       │   │       └── terragrunt.hcl
│       │   └── env.hcl
│       └── region.hcl
└── modules
```

### 2 or more teams  

You can have multiple operational teams. One for the network, IAM or even data engineers.

➡️ Create a repository for each team to separate git flows

➡️ You can then follow the guidelines for `1 team` for each team.
