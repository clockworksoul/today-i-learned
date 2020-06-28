# Terraform provider for generic REST APIs

_Source: [Github](https://github.com/Mastercard/terraform-provider-restapi)_

The Terraform provider for generic REST APIs allows you to interact with APIs that may not yet have a first-class provider available.

There are a few requirements about how the API must work for this provider to be able to do its thing:

* The API is expected to support the following HTTP methods:
    * POST: create an object
    * GET: read an object
    * PUT: update an object
    * DELETE: remove an object

* An "object" in the API has a unique identifier the API will return

* Objects live under a distinct path such that for the path `/api/v1/things`...
    * POST on `/api/v1/things` creates a new object
    * GET, PUT and DELETE on `/api/v1/things/{id}` manages an existing object

Full documentation is available at https://github.com/Mastercard/terraform-provider-restapi.

## Examples

In this example, we are using the `fakeserver` available with this provider to create and manage imaginary users in [an imaginary API server defined here](https://github.com/Mastercard/terraform-provider-restapi/tree/master/fakeserver).

### Start and initialize the dummy `fakeserver`

To use this example fully, start up `fakeserver` and add some dummy data.

```bash
$ curl 127.0.0.1:8080/api/objects -X POST -d '{ "id": "8877", "first": "John", "last": "Doe" }'
$ curl 127.0.0.1:8080/api/objects -X POST -d '{ "id": "4433", "first": "Dave", "last": "Roe" }'
```

After running `terraform apply`, you will now see three objects in the API server:

```bash
$ curl 127.0.0.1:8080/api/objects | jq
```

### Configure the provider

```hcl
provider "restapi" {
  uri                  = "http://127.0.0.1:8080/"
  debug                = true
  write_returns_object = true
}
```

### `GET` an object via a data source

This will make information about the user named "John Doe" available by finding him by first name.

```hcl
data "restapi_object" "John" {
  path         = "/api/objects"
  search_key   = "first"
  search_value = "John"
}
```

### Add an object via `terraform import`

You can import the existing Dave Roe resource from a file by executing:

```bash
$ terraform import restapi_object.Dave /api/objects/4433
```

### Manage it here, too!

The `import` will pull in Dave Roe, but the subsequent `terraform apply` will change the record to "Dave Boe"

```hcl
resource "restapi_object" "Dave" {
  path = "/api/objects"
  data = "{ \"id\": \"4433\", \"first\": \"Dave\", \"last\": \"Boe\" }"
}
```

### Add a resource

This will ADD the user named "Foo" as a managed resource

```hcl
resource "restapi_object" "Foo" {
  path = "/api/objects"
  data = "{ \"id\": \"1234\", \"first\": \"Foo\", \"last\": \"Bar\" }"
}
```

### Variable interpolation

Congrats to Jane and John! They got married. Give them the same last name by using variable interpolation

```hcl
resource "restapi_object" "Jane" {
  path = "/api/objects"
  data = "{ \"id\": \"7788\", \"first\": \"Jane\", \"last\": \"${data.restapi_object.John.api_data.last}\" }"
}
```
