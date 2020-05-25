# Terraform Custom Variable Validation Rules

_Sources: [[1]](https://github.com/hashicorp/terraform/issues/2847#issuecomment-573252616), [[2]](https://www.terraform.io/docs/configuration/variables.html#custom-validation-rules)_

_**Note: As of today (25 May 2020) this feature is considered experimental and is subject to breaking changes even in minor releases. We welcome your feedback, but cannot recommend using this feature in production modules yet.**_

In Terraform, a module author can specify arbitrary custom validation rules for a particular variable using a `validation` block nested within the corresponding `variable` block:

```hcl
variable "image_id" {
  type        = string
  description = "The id of the machine image (AMI) to use for the server."

  validation {
    condition     = length(var.image_id) > 4 && substr(var.image_id, 0, 4) == "ami-"
    error_message = "The image_id value must be a valid AMI id, starting with \"ami-\"."
  }
}
```

The `condition` argument:

* Must use the value of the variable,
* Must return `true` if the value is valid, or `false` if it is invalid,
* May refer only to the variable that the condition applies to, and
* Must not produce errors.

If the failure of an expression is the basis of the validation decision, use [the `can` function](https://www.terraform.io/docs/configuration/functions/can.html) to detect such errors. For example:

```hcl
variable "image_id" {
  type        = string
  description = "The id of the machine image (AMI) to use for the server."

  validation {
    # regex(...) fails if it cannot find a match
    condition     = can(regex("^ami-", var.image_id))
    error_message = "The image_id value must be a valid AMI id, starting with \"ami-\"."
  }
}
```

If `condition` evaluates to `false`, Terraform will produce an error message that includes the sentences given in `error_message`. The error message string should be at least one full sentence explaining the constraint that failed, using a sentence structure similar to the above examples.

This is an experimental language feature that currently requires an explicit opt-in using the experiment keyword `variable_validation`:

```hcl
terraform {
  experiments = [variable_validation]
}
```
