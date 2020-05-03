# Terraform Plugin Basics

_Sources: https://www.terraform.io/docs/plugins/basics.html_

This page documents the basics of how the plugin system in Terraform works, and how to setup a basic development environment for plugin development if you're writing a Terraform plugin.

## How it Works

Terraform providers and provisioners are provided via plugins. Each plugin exposes an implementation for a specific service, such as AWS, or provisioner, such as bash.

Plugins are executed as a separate process and communicate with the main Terraform binary over an RPC interface.

More details are available in [Plugin Internals](https://www.terraform.io/docs/internals/internal-plugins.html).

## Installing Plugins

The provider plugins distributed by HashiCorp are automatically installed by `terraform init`. Third-party plugins (both providers and provisioners) can be manually installed into the user plugins directory, located at %APPDATA%\terraform.d\plugins on Windows and ~/.terraform.d/plugins on other systems.

For more information, see:

* [Configuring Providers](https://www.terraform.io/docs/configuration/providers.html)
* [Configuring Providers: Third-party Plugins](https://www.terraform.io/docs/configuration/providers.html#third-party-plugins)

For developer-centric documentation, see:

* [How Terraform Works: Plugin Discovery](https://www.terraform.io/docs/extend/how-terraform-works.html#discovery)

## Developing a Teraform Plugin

Developing a plugin is simple. The only knowledge necessary to write a plugin is basic command-line skills and basic knowledge of the [Go programming language](https://golang.org/).

Create a new Go project. The name of the project should either begin with `provider-` or `provisioner-`, depending on what kind of plugin it will be. The repository name will, by default, be the name of the binary produced by go install for your plugin package.

With the package directory made, create a `main.go` file. This project will be a binary so the package is "main":

```go
package main

import (
    "github.com/hashicorp/terraform/plugin"
)

func main() {
    plugin.Serve(new(MyPlugin))
}
```

The name `MyPlugin` is a placeholder for the struct type that represents your plugin's implementation. This must implement either `terraform.ResourceProvider` or `terraform.ResourceProvisioner`, depending on the plugin type.

To test your plugin, the easiest method is to copy your terraform binary to `$GOPATH/bin` and ensure that this copy is the one being used for testing. `terraform init` will search for plugins within the same directory as the `terraform` binary, and `$GOPATH/bin` is the directory into which `go install` will place the plugin executable.

Next Page: [Terraform Provider Plugin Development](provider-plugin-development.md)
