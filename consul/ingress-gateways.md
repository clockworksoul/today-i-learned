# Ingress Gateways in HashiCorp Consul 1.8

_Source: [Ingress Gateways in HashiCorp Consul 1.8](https://www.hashicorp.com/blog/ingress-gateways-in-hashicorp-consul-1-8/), [HashiCorp Learn](https://learn.hashicorp.com/consul/developer-mesh/ingress-gateways)_

Consul ingress gateways provide an easy and secure way for external services to communicate with services inside the Consul service mesh.

![Consul Ingress Gateway](https://www.datocms-assets.com/2885/1592432846-ingress-gateways-aws.png?w=500)

Using the Consul CLI or our Consul Kubernetes Helm chart you can easily set up multiple instances of ingress gateways and configure and secure traffic flows natively via our Layer 7 routing and intentions features.

# Overview

Applications that reside outside of a service mesh often need to communicate with applications within the mesh. This is typically done by providing a gateway that allows the traffic to the upstream service. Consul 1.8 allows us to create an ingress solution without us having to roll our own.

# Using Ingress Gateways on VMs

Let’s take a look at how Ingress Gateways work in practice. First, let’s register our Ingress Gateway on port 8888:

```bash
$ consul connect envoy -gateway=ingress -register -service ingress-gateway \
  -address '{{ GetInterfaceIP "eth0" }}:8888'
```

Next, let’s create and apply an ingress-gateway [configuration entry](https://www.consul.io/docs/agent/config-entries/ingress-gateway) that defines a set of listeners that expose the desired backing services:

```hcl
Kind = "ingress-gateway"
Name = "ingress-gateway"
Listeners = [
  {
    Port = 80
    Protocol = "http"
    Services = [
      {
        Name = "api"
      }
    ]
  }
]
```

With the ingress gateway now configured to listen for a service on port 80, we can now route traffic destined for specific paths to that specific service listening on port 80 of the gateway via a [service-router](https://www.consul.io/docs/agent/config-entries/service-router) config entry:

```hcl
Kind = "service-router"
Name = "api"
Routes = [
  {
    Match {
      HTTP {
        PathPrefix = "/api_service1"
      }
    }

    Destination {
      Service = "api_service1"
    }
  },
  {
    Match {
      HTTP {
        PathPrefix = "/api_service2"
      }
    }

    Destination {
      Service = "api_service2"
    }
  }
]
```

Finally, to secure traffic from the Ingress Gateway, we can also then define intentions to allow the Ingress Gateway to communicate to specific upstream services:

```bash
$ consul intention create -allow ingress-gateway api_service1
$ consul intention create -allow ingress-gateway api_service2
```

Since Consul also specializes in Service Discovery, we can also discovery gateways for Ingress Gateway enabled services via DNS using [built-in subdomains](https://www.consul.io/docs/agent/dns#ingress-service-lookups):

```bash
$ dig @127.0.0.1 -p 8600 api_service1.ingress.<datacenter>.<domain>. ANY
$ dig @127.0.0.1 -p 8600 api_service2.ingress.<datacenter>.<domain>. ANY
```
