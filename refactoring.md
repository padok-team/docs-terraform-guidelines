# Refactoring

## Why do you need refactoring ?

Refactoring your codebase terraform is a necessity within the lifecycle of your codebase.
As your infrastructure grows and as you add collaborators to the project, you'll need to reconsider things like :

- the size of the layers
- the scope of the modules
- the interface of the modules

To keep track of your continuous changes, you should set up "quality probes", which will serve as goals the refactored codebase aims to reach like :

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

- Less than 2 *aka you and a partner*.

  You should frequently work in pair and communicate on the refacto. You must review each others PR, that way both of you will have the entire knowledge of what's merged into the codebase.

- 3 collaborators and more *aka a team*.

  What's risky with a team refactoring a codebase is poor quality code merged into the codebase due to a loss in information transmission. To control code quality over time during the refactoring you need to proceed as follows :

  - Write out the new standards you want to implement
  - Dedicate time on formation for members of the team on the new standards
  - Archive your current codebase
  - Then start implementing on a dedicated codebase

  That way you'll have a clear view on your refactored code. Also this pin in time your codebase state, not your infrastructure state. This tag is meant to track modifications not to be used again to apply.

### Do you have splitted layers ? #Accelerate

The anti-pattern of this is having **only one terraform state** with every resources in it. This slows down the pace of feature promotion.

To know how to define the scope of the new layers, you can ask yourself how to dispatch resources **in 3 states** or also in 3 folders. You can rely on the 3-tier way of splitting an architecture. If you realize while refactoring that terraform plans takes too long (more than 1 minute), you may need to split it again.

The gains would be :

- Reduce time spent on terraform plan and apply
- To decouple resources with different update frequency (for instance : **DNS records** may be updated frequently to add new endpoints whereas **VPC** are created once and never updated)
- Anticipate the scope definition of your yet to be written modules

### Do you have modules ? #DRY

Modules serve 1 purpose : Don't repeat yourself. You write modules for 2 reasons :

- Wrap multiple resources under 1 logic bloc
- Hide complexity from layers.

Use as much modules from the Padok's library or Providers repositories as you can. If no module there matches you needs, here is how you should implement modules in your codebase :

- Identify similar resources created between your environments
- Spot the differences in configuration between them
- Put resources in the module leaving only the necessary configuration in the interface

Pay attention to terraform plan elapsed time as you build your modules.

### Are your resources compliant with naming, versioning and other standards ? #CleanCode

Once you have your modules, you should focus on code readability and maintainability. So, implement [naming standards](./terraform_naming.md), [versioning standards](./terraform_versioning.md) and others.

### Is your codebase growing ? Is your project in build phase ?

You should integrate refactoring alongside infrastructure feature delivery. It's more confortable to refactor a cold codebase that stands here for a living infrastructure but most of the time the codebase is 'hot' which means it's used to deploy new infrastructure features.

Following the plan above, you can build up a consitent backlog. Manage to include refactoring as an epic on it's own. If delivery pressure is intense, keep at least 20% of you velocity on refactoring. Else, if your estimation on the Epic exceed 3 sprints (in agile terms), You should consider allowing it 50% of your velocity.

You'll be able to see the benefits of your refactoring by continuously tracking the maximum time a plan can take.

Refactoring your codebase from within could be easier if you use the [tfautomv](https://tfautomv.dev/) tool as it handles for you the changes in the terraform states for identical unchanged resources.

## Don'ts

- Trying to change mutliple parts or make multiple steps at once.
  
  It's very tempting to change a parameter on the resource or bump some versions while migrating to modules. But taking small steps and splitting complexity is always a better idea. Proceed at slow pace but keep the plan clear.

- Split in too many layers

  This creates incomprehensible codebase, the information is spread across so many files that you won't have a reliable comprehension of what your infrastructure stack is made of.
