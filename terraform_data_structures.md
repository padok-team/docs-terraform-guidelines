# Terraform Data Structures

This standard disposes recommendations about the structures to use in Terraform code, and why.

## TLDR;
> Promoting usage of map instead of arrays in modules interfaces.

## List or map?

### Usage and limits of lists

The bigger is an infrastructure, the more optimized would be the underlaying code.

Use array and a count in Terraform is a solution to avoid repetition when creating a lot of resources.

```yaml
variable "public_dns" {
    type = list({
        name    = string
        records = list(string)
        type    = string
    })
}

resource "aws_route53_record" "public-dns" {
  count           = length(var.public_dns)
  zone_id         = var.public_dns_zone_id
  name            = var.public_dns[count.index].name
  type            = var.public_dns[count.index].type
  records         = var.public_dns[count.index].records
  ttl             = "90"
}
```

And it also create in the state a list of resources identified by a numeric key.

```bash
$ tf state list
aws_route53_record.public-dns[0]
aws_route53_record.public-dns[1]
aws_route53_record.public-dns[2]
aws_route53_record.public-dns[3]
...
```

In a list of parameters given as an array, the order is important. Each index must be continuous. Therefore, when we have to delete a resources, 3 options offers to us:

* **Simply deleting the value**. The rest of resources will be recreated after. It can be mostly used for stateless resources, such as configuration, access policies, etc.
* **Deleting the value and passing a new one instead to keep the order**. For a lot of reasons, this is a bad idea. Trying to keep an order just to avoid recreation of resources gives a lost of clarity in the list of values (for exemple, when they're ordered by alphabetical order)
* **Deleting the value and remap the others in state**. Manipulation of state should be done as less as possible to guarantee coherence between infrastructure, it's representation, and code. This last one is just a hack and should never be used.

### Usage and recommandations for using maps

Accessing maps values are done by the use of a `key`, not an `index`. The difference is that an `index` have to be continuous, while a `key` doesn't have to be ordered.

Also, the `map` use the `for_each` function, which allows to use the `each` iterator. The code is lighter than before when accessing values.

```yaml
variable "public_dns" {
    type = map({
        records = list(string)
        type    = string
    })
}

resource "aws_route53_record" "public-dns" {
  for_each        = var.public_dns
  zone_id         = var.public_dns_zone_id
  name            = each.key
  type            = each.value.type
  records         = each.value.records
  ttl             = "90"
}
```

A state will use a string instead of a number for the identification. This is more clear when reviewing state resources, when controlling a plan, etc.

```bash
$ tf state list
aws_route53_record.public-dns["example.org"]
aws_route53_record.public-dns["webmail.example.org"]
aws_route53_record.public-dns["other.example.org"]
aws_route53_record.public-dns["extranet.example.org"]
...
```

When manipulating values, in the particular case of deletion, this allows to delete only the desired resource, without recreating each next value afterwards.
