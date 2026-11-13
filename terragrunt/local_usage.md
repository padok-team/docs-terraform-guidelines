# Local usage

In Terragrunt files, `locals` are a convenient way to name a value. Used without a reason, they only add a level of indirection.

## TL;DR

> Define a local only if it is reused (DRY) or if it abstracts complexity.
> Otherwise, inline its value where it is used.

## Define a local only when it is useful

A local is useful in two cases:

- **DRY**: the value is used more than once in the file. Changing it in one place updates every usage.
- **Complexity**: the local merges several values or encapsulates an expression, and its name explains the intent.

A local that holds a literal or an alias to another value, and is used only once, is neither of those.

### Why?

- **Readability**
  - To understand a value, the reader has to jump to the `locals` block and back. For a value used once, inlining is faster to read.
- **Maintainability**
  - Every local adds a name to keep in sync with the value it holds. Unused or single-use locals accumulate as the code evolves.
- **Signal**
  - When every local has a reason to exist, a reader knows a local is shared or hides something complex, without having to check.

### Recommendation

Inline values that are used once and need no explanation.

Not recommended ❌

```hcl
locals {
  region = "eu-west-3" # used once, nothing to abstract
}

inputs = {
  region = local.region
}
```

Ok :+1:

```hcl
inputs = {
  region = "eu-west-3"
}
```

Define a local when the value is reused or hides complexity.

Ok :+1:

```hcl
locals {
  # Reused: defined once, used in several places
  environment = basename(get_original_terragrunt_dir())

  # Complexity: merges several values behind a meaningful name
  tags = merge(
    local.root.locals.common_tags,
    { environment = local.environment },
  )
}

inputs = {
  tags        = local.tags
  bucket_name = "my-project-${local.environment}"
}
```

If a value is repeated across several layers rather than within a file, a local is not the answer: move the input up the tree instead (see the [Context pattern](context_pattern.md)).
