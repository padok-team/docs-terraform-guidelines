# Creating a good IaC structure

## Objective

This documentation aims to help you choose the best code structure for your project

## Flow

<details><summary>Are you creating an infrastructure from scratch ? </summary>  
  
Great ! First, how many teams will work on this infrastructure ?

<details><summary>1 team (The operational team)</summary>  

Only the operational team will be creating components in this infrastructure. Make sure that you know how will devs (and possibly data engineer) deploy their code and who creates what they will need to host them.

How many segmentation do you need to create your resources ? (Sandbox, dev, staging, production or is it global to all environments)

<details><summary>1 environment or multi environment ressources</summary>  

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

- Wizzair

</details>

<details><summary>2 or + environment</summary>  

This is the most classic setup, having multiple environment to have segmentation between stages of release.

To organize your code you will need to reflect on what you are most likely to create regularly ?

<details><summary>Environment (default approach)</summary>  

This is the default approach. You can start with pre-production and production.
This can also be used if you don't need Terraform to create the application requirement (Ex: For every app I need a new database or ECS instance)

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

</details>

<details><summary>Applications</summary>  

This approach gives more responsibility on each application

➡️ IaC application oriented

```tree
.
├── application-1
│   ├── app.hcl
│   └── fr
│       ├── dev
│       │   ├── bucket
│       │   │   └── terragrunt.hcl
│       │   ├── database
│       │   │   └── terragrunt.hcl
│       │   ├── ecs
│       │   │   └── terragrunt.hcl
│       │   └── env.hcl
│       └── region.hcl
└── application-2
    ├── app.hcl
    └── fr
        ├── dev
        │   ├── bucket
        │   │   └── terragrunt.hcl
        │   └── env.hcl
        └── region.hcl
```
</details>
</details>
</details>
<details><summary>2 or more teams</summary>  

You can have multiple operational teams. One for the network, IAM or even data engineers.

➡️ Create a repository for each team to seperate git flows

➡️ You can then follow the guidelines for `1 team` for each team.

</details>
</details>


<br>
<details>  
<summary> Are you contributing to an existing code base ? </summary>  
  
TODO

<details>  
<summary> Have you identified where you will add your ressources ? </summary>  
    

    
</details>
</details>
