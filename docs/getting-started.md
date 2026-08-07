# Getting started

From an empty directory to a first managed resource. Two paths: Terraform if you
want infrastructure as code, Go if you are building something of your own.

## Get a token

Everything authenticates with a permanent API token issued in the EdgeCenter
control panel. One token works across cloud, CDN, DNS, storage, protection and
monitoring.

```sh
export EC_PERMANENT_TOKEN="your-token"
```

Keep it out of committed files. The Terraform provider picks the variable up on
its own, and every SDK accepts the token through a client option.

You also need the numeric **project** and **region** identifiers for cloud
resources. Both are visible in the control panel and through the API.

## Terraform

The provider is published in the public registry as `Edge-Center/edgecenter`.

```hcl
terraform {
  required_providers {
    edgecenter = {
      source  = "Edge-Center/edgecenter"
      version = "~> 0.14"
    }
  }
}

provider "edgecenter" {}

resource "edgecenter_network" "example" {
  name       = "example"
  type       = "vxlan"
  project_id = 1
  region_id  = 1
}

resource "edgecenter_subnet" "example" {
  name            = "example"
  cidr            = "192.168.10.0/24"
  network_id      = edgecenter_network.example.id
  dns_nameservers = ["8.8.8.8", "1.1.1.1"]
  project_id      = 1
  region_id       = 1
}
```

```sh
terraform init
terraform plan
terraform apply
```

The empty `provider "edgecenter" {}` block reads `EC_PERMANENT_TOKEN` from the
environment. Pass `permanent_api_token` explicitly only when you have a reason to.

Reference for all 62 resources and 44 data sources is on the
[Terraform Registry](https://registry.terraform.io/providers/Edge-Center/edgecenter/latest/docs).

### Two things that surprise people

**State holds secrets.** Terraform writes whatever the API returns into the state
file, including generated credentials. Treat the state as sensitive: use a remote
backend with encryption, and never commit `terraform.tfstate`.

**Project and region are per-resource.** They are not global provider settings for
every resource type. Cloud resources take `project_id` and `region_id`; CDN, DNS
and storage resources do not, because those services are not regional.

## Go

Each service is a separate module. Take only the ones you need.

```sh
go get github.com/Edge-Center/edgecentercloud-go/v2
```

```go
package main

import (
	"context"
	"fmt"
	"os"

	edgecloud "github.com/Edge-Center/edgecentercloud-go/v2"
)

func main() {
	client, err := edgecloud.NewWithRetries(nil,
		edgecloud.SetAPIKey(os.Getenv("EC_PERMANENT_TOKEN")),
		edgecloud.SetRegion(1),
		edgecloud.SetProject(1),
	)
	if err != nil {
		panic(err)
	}

	instances, _, err := client.Instances.List(context.Background(), nil)
	if err != nil {
		panic(err)
	}

	for _, i := range instances {
		fmt.Println(i.ID, i.Name, i.Status)
	}
}
```

`NewWithRetries` is the constructor to reach for by default: it retries on the
transient failures the platform produces under load. `New` gives you a client
without that behaviour when you want to own the retry policy.

The client aggregates roughly thirty typed services as fields - `Instances`,
`Networks`, `Loadbalancers`, `Volumes`, `MkaaS` and the rest. Each method returns
the decoded value, the raw `*Response`, and an error.

### The other SDKs

`edgecentercdn-go` and `edgecenteredgemon-go` share a different shape: a
`Requester` transport core in the `edgecenter` package, one sub-package per
resource, and typed errors with sentinels such as `ErrNotFound` and `ErrConflict`
you can match with `errors.Is`. Construct the client once and pass it to the
resource packages you need.

`edgecenter-dns-sdk-go` and `edgecenter-storage-sdk-go` are older and flatter, and
still target Go 1.19.

See the [repository map](repository-map.md) for what each module covers and
[compatibility](compatibility.md) for versions.

## Kubernetes

Running a cluster on EdgeCenter Cloud usually means three components:

- [edgecenter-ccm](https://github.com/Edge-Center/edgecenter-ccm) for node
  lifecycle and `type: LoadBalancer` services.
- [edgecenter-csi](https://github.com/Edge-Center/edgecenter-csi) for persistent
  volumes backed by EdgeCenter Disks, `ReadWriteOnce`, driver name
  `csi.edgecenter.org`, Kubernetes 1.31 or newer.
- [external-dns-ec-webhook](https://github.com/Edge-Center/external-dns-ec-webhook)
  if you want DNS records to follow your ingresses.

Charts come from the organization Helm repository:

```sh
helm repo add edgecenter https://edge-center.github.io/ec-helm-charts.github.io/
helm repo update
helm search repo edgecenter
```

If you use MKaaS, the managed Kubernetes service, the cluster arrives with these
components already wired and you manage it through the
`edgecenter_mkaas_cluster` and `edgecenter_mkaas_pool` Terraform resources
instead.

## Where to go next

- [Repository map](repository-map.md) - the full inventory.
- [Development](development.md) - if you plan to change any of this code.
- [API reference](https://apidocs.edgecenter.ru) - the endpoints underneath.
