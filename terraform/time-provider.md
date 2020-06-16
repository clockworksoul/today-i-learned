# Terraform Time Provider

_Source: [The Terraform Docs](https://www.terraform.io/docs/providers/time/index.html)

The time provider is used to interact with time-based resources. The provider itself has no configuration options.

## Resource "Triggers"

Certain time resources, only perform actions during specific lifecycle actions:

* `time_offset`: Saves base timestamp into Terraform state only when created.
* `time_sleep`: Sleeps when created and/or destroyed.
* `time_static`: Saves base timestamp into Terraform state only when created.

These resources provide an optional map argument called `triggers` that can be populated with arbitrary key/value pairs. When the keys or values of this argument are updated, Terraform will re-perform the desired action, such as updating the base timestamp or sleeping again.

For example:

```hcl
resource "time_static" "ami_update" {
  triggers = {
    # Save the time each switch of an AMI id
    ami_id = data.aws_ami.example.id
  }
}

resource "aws_instance" "server" {
  # Read the AMI id "through" the time_static resource to ensure that
  # both will change together.
  ami = time_static.ami_update.triggers.ami_id

  tags = {
    AmiUpdateTime = time_static.ami_update.rfc3339
  }

  # ... (other aws_instance arguments) ...
}
```

`triggers` are _not_ treated as sensitive attributes; a value used for `triggers` will be displayed in Terraform UI output as plaintext.

To force a these actions to reoccur without updating `triggers`, the `terraform taint` command can be used to produce the action on the next run.