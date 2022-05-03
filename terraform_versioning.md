# Terraform Versioning

This settles the terraform and providers version management.
It does not discuss the terraform module version management.

## TLDR;
> *Be flexible about versions in modules; be unambiguous about versions in
> layers.*

## Environnement layer versioning

### Terraform < 1.0

At the root of a layer (ie, the directory where "terraform apply" is run),
best practice is to specify an exact version of Terraform to use. This is due
to terraform state incompatibility between 0.X versions. Use the
`= 0.2.3` constraint to do this.

> For more information: <https://www.terraform.io/docs/language/settings/index.html#specifying-a-required-terraform-version>

```yaml
terraform {
  required_version = "0.14.0"
  }
```

### Terraform > 1.0 and < 2.0

In terraform `1.x` and above, even though terraform guarantees no breaking
changes on its state, you should stick with specifying an exact version of
Terraform to use. Use the `= 1.0.0` constraint to do this.

> For more information: <https://www.terraform.io/docs/language/settings/index.html#specifying-a-required-terraform-version>

```yaml
terraform {
  required_version = "1.0.0"
}
```

### Required providers version for layers

If the layer uses providers directly, as opposed to only through modules, then
you should also specify version contraints for those providers in the layer's
configuration. 

In order to make the behaviour of layers completely deterministic, the version
should be fixed to a specific version. Use the `= 1.0.0` constraint to do this.

## Bump terraform version

TODO: List steps like

1. Review Changelog
2. Make modifications
3. If possible, `terraform plan`
4. Change terraform version
5. tfswitch
6. `terraform plan`
7. `terraform apply`
8. ...

## Module layer versioning

### Terraform CLI version

In a module, you can allow more flexibility with regards to Terraform's
minor and/or patch versions. For example, the `~> 1.0` constraint will allow
all 1.x.x versions of Terraform, while the `~> 1.0.0` constraint will allow
all 1.0.x versions.

**Modules that use features added in a specific minor version (eg: moved blocks added in 1.1) should require that version as a minimum.**

> For more information: <https://www.terraform.io/docs/language/settings/index.html#specifying-a-required-terraform-version>

```yaml
terraform {
  required_version = "~> 1.0"
}
```

### Required providers version for modules

You can indicate that the module requires that certain providers are configured by the caller.
Again; use the "~> 1.0" constraint to allow all minor and/or patch versions.

> For more information: <https://www.terraform.io/docs/language/providers/requirements.html>

```yaml
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.12"
    }
    google-beta = {
      source  = "hashicorp/google-beta"
      version = "~> 4.12"
    }
    mongodbatlas = {
      source  = "mongodb/mongodbatlas"
      version = "~> 1.3"
    }
  }
}
```

## Bump version

TODO
