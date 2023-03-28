# Handling secrets' values with Terraform

![Use cases for secrets handling](../assets/img/Handling%20secrets%20with%20Terraform.png)

## TL;DR

WIP

## Pre-requsites

### Secure access to your Terraform state

Keep in mind that Terarform state is a sensitive file. It contains all the information about your infrastructure. It's a good practice to store it in a secure place. You can use a private backend storage with very limited access to the only people / services that need to access it _(e.g. Terraform Cloud, Terraform Enterprise, S3, GCS, etc.)_.

Also, you should use encryption on your backend storage since most of them support it.

If your backend storage does not support encryption, then, maybe you should considerer using a third-party encryption tool like [sops](https://registry.terraform.io/providers/carlpett/sops/latest/docs) or even use a Secret Management Tool _(c.f. bellow for explanations)_.

## Recommended way to handle secrets

### Use a [`random_password`](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password) resource

[This resource](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password) looklike the `random_string` one but the difference is that the result is treated as sensitive and will not be displayed in the console output.

```json
resource "random_password" "password" {
    length           = 16
    special          = true
    override_special = "!#$%&*()-_=+[]{}<>:?"
}

resource "aws_db_instance" "example" {
    instance_class    = "db.t3.micro"
    allocated_storage = 64
    engine            = "mysql"
    username          = "someone"
    password          = random_password.password.result
}
```

### Use a Secret Management Tool

Also, you can store this sensitive data in a Secret Management Tool _(e.g. AWS Secrets Manager, HashiCorp Vault, etc.)_.
