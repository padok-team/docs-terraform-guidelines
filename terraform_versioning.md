# Terraform Versioning

## Environnement layer versioning

At the root of a layer (ie, the directory where "terraform apply" is run),
best practice is to specify an exact version of Terraform to use. Use the
"= 1.2.3" constraint to do this.

> For more information: <https://www.terraform.io/docs/language/settings/index.html#specifying-a-required-terraform-version>

```yaml
terraform {
  required_version = "= 1.1.8"
}
```

## Module layer versioning

### Terraform CLI version

### Required providers version

In a module, you can allow more flexibility with regards to Terraform's
minor and/or patch versions. For example, the "~> 1.0" constraint will allow
all 1.x.x versions of Terraform, while the "~> 1.0.0" constraint will allow
all 1.0.x versions.

> For more information: <https://www.terraform.io/docs/language/settings/index.html#specifying-a-required-terraform-version>

```yaml
terraform {
  required_version = "~> 1.0"
}
```

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
