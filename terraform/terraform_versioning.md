# Terraform Versioning

This settles the terraform and providers version management.
It does not discuss the terraform module version management.

## TLDR;
> *Be flexible about versions in modules; be unambiguous about versions in
> layers.*

## Environnement layer versioning

### Terraform < 1.0

At the root of a layer (ie, the directory where “terraform apply” is run),
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
you should also specify version contraints for those providers in the layer’s
configuration. 

In order to make the behaviour of layers completely deterministic, the version
should be fixed to a specific version. Use the `= 1.0.0` constraint to do this.

### Terraform lock files in layers

Lock files in Terraform are generated upon running `terraform init`. These
files, named `.terraform.lock.hcl`, must be committed into the repo. They allow
for deterministic behaviour in dependency management across devices running our
code.

These files only appear in directories in which you run `terraform init`,
therefore there are none in modules.

## Bump terraform version

First of all, you have to review the [Terraform Changelog](https://github.com/hashicorp/terraform/blob/main/CHANGELOG.md) to identify changes that may affect the behavior of your code. As a recent example, the `optional` function was experimental, and passed standard in 1.3. It causes codes to be remanied to remove the declaration of an experimental feature, in the layer and used submodules.

You will have to change the version of your layer during this, that have to be done in your layer only ([modules shouldn’t have fixed versions](#module-layer-versioning)). Change also your Terraform CLI version, you can use tools such as `tfswitch`.

Perform a plan is okay, but don’t apply during the phase of adaptation. You don’t want to create any drift by upgrading your Terraform version. As the version is stored in the state file, under `terraform_version`, experimenting an upgrade and applying may lead to undesired conflicts of versions due to the state. Note: this field is automaticly updated when applying with a new Terraform version, even if Terraform output just displays that the infrastructure matches the configuration.

Finally, if your Terraform plan doesn’t see any differences with the actual code, you can commit your code, then applying when merged on the principal branch. This way is impactless for your coworkers or a CI, because the first who will apply the latest code will perform the migration in the state.

## Module layer versioning

### Terraform CLI version

In a module, you can allow more flexibility with regards to Terraform’s
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
Again; use the ”~> 1.0” constraint to allow all minor and/or patch versions.

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
