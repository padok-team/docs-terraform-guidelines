# Refactoring

## Why do you need refactoring ?

**Refactoring** your codebase terraform is a **necessity** within the lifecycle of your codebase.
As your **infrastructure grows** and as you add collaborators to the project, you'll need to reconsider things like :

- the size of the layers
- the scope of the modules
- the interface of the modules

To keep track of your continuous changes, you should set up _quality probes_, which will serve as goals the refactored codebase aims to reach like :

- the maximum time a plan can take
- the number of variables you need to provide before implementing the layer
- the time it takes to implement your 3 most frequent tasks.

## Questions to ask yourself

Your priorities regarding a refactoring are the following in this order :

1. Control code quality
2. Accelerate feature promotion
3. Reduce code duplication
4. Improve readability

![refacto_decision_tree](assets/img/refacto_decision_tree.png)

### How many collaborators are contributing to the codebase ? #CodeQualityControl

- Less than 2 _aka you and a partner_.

  You should frequently **work in pair** and communicate on the refacto. You must **review each others PR**, that way both of you will have the entire knowledge of what's merged into the codebase.

- 3 collaborators and more _aka a team_.

  What's risky with a team refactoring a codebase is **poor quality code merged** into the codebase due to a loss in information transmission. To **control code quality** over time during the refactoring you need to proceed as follows :

  - Write out the **new standards** you want to implement
  - Dedicate time on **training** for members of the team on the new standards
  - Put your current codebase on hold as **legacy**
    > Permit yourself to edit your legacy codebase only if you do not create `resource` bloc.
    > It's OK if you need like a new buckets which are created looping over a list of string.
    > If you need to **copy paste** the `resource` bloc or **write a new one** -> do it in the **new codebase**.And when you do, **don't take legacy codebase in example**
  - Then start implementing on a dedicated codebase

  That way you'll have a clear view on your refactored code. Also this pin in time your codebase state, not your infrastructure state. This tag is meant to track modifications not to be used again to apply.

### Do you have splitted layers ? #Accelerate

The goal is to find the right balance between **macro layers** (too few layers) and **micro layers** (too many layers).

**Macro layers** (too few, ideally avoid having only one terraform state with every resource in it) create problems such as:

- Slow terraform plan and apply operations
- Difficulty in feature promotion
- Coupling of resources with different update frequencies
- Increased blast radius for changes

**Micro layers** (too many) create different problems such as:

- Incomprehensible codebase with information spread across too many files
- Loss of overview of the overall infrastructure stack
- Increased complexity in managing dependencies between layers

To know how to define the **scope of balanced layers**, you can ask yourself how to dispatch resources **in 3 states** or also in 3 folders. You can rely on the 3-tier way of splitting an architecture. If you realize while refactoring that terraform plans takes too long (_more than 1 minute_), you may need to split it again.

The gains would be :

- **Reduce time** spent on terraform plan and apply
- To decouple resources with different update frequency (for instance : **DNS records** may be updated frequently to add new endpoints whereas **VPC** are created once and never updated)
- Anticipate the scope definition of your yet to be written modules

### Do you have modules ? #DRY

Modules serve 1 purpose : Don't repeat yourself. You write modules for 2 reasons :

- Wrap multiple resources under **1 logic bloc**
- Hide complexity from layers.

Use as much modules from the Theodo Cloud internal library or Providers repositories as you can. If no module there matches your needs, here is how you should implement modules in your codebase:

- Identify similar resources created between your environments
- Spot the differences in configuration between them
- Put resources in the module leaving only the necessary configuration in the interface

Pay attention to terraform plan elapsed time as you build your modules.

### Are your resources compliant with naming, versioning and other standards ? #CleanCode

Once you have your modules, you should focus on code readability and maintainability. So, implement [naming standards](./terraform_naming.md), [versioning standards](./terraform_versioning.md) and others.

## Don'ts

- Trying to change multiple parts or make multiple steps at once.

  It's very tempting to change a parameter on the resource or bump some versions while migrating to modules. But taking small steps and splitting complexity is always a better idea. Proceed at slow pace but keep the plan clear.

- Going to extremes with layer design

  Avoid both **macro layers** (everything in one state) and **micro layers** (excessive fragmentation). Find the right balance for your specific use case and team size.
