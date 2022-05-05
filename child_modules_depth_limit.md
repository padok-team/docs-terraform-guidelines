# Terraform modules childs limits

We've all seen a root module with a child module that also has a child module.

## TL;DR

Make sure you have no more than 2 levels in your module hierarchy.

## Limit the depth of the your modules' tree

We do not recommand to have a module tree that is too deep.

### Why?

* **Debugging**
  * It is hard to understand what is going on in the code -> debugging is hard
* **Maintainability**
  * Upgrade a root, child or grandchild modules may have an impact on the whole module tree

### Recommandation

We recommand to have a module tree that is no more than 2 levels deep like the following tree:

Ok :+1:

```yaml
├── root_module # <- Level 0
│   ├── child_module_1 # <- Level 1
│   │   ├── grandchild_module_1 # <- Level 2
│   ├── child_module_2 # <- Level 1
```

Not ok :-1:

```yaml
├── root_module # <- Level 0
│   ├── child_module_1 # <- Level 1
│   │   ├── grandchild_module_1 # <- Level 2
│   │   │   ├── grand_grandchild_module_1 # <- Level 3
│   ├── child_module_2 # <- Level 1
```
