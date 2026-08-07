# Repository map

Every public repository in the [Edge-Center](https://github.com/Edge-Center)
organization, what it is for, and how much movement it sees. Activity is stated
as the year of the last change rather than a promise about the future.

## Terraform

### terraform-provider-edgecenter

The flagship, and the only repository here that most users ever touch directly.
62 resources and 44 data sources covering cloud compute and networking, load
balancers, MKaaS, DBaaS, CDN, DNS, Object Storage, DDoS Protection and RMON
monitoring.

- Module: `github.com/Edge-Center/terraform-provider-edgecenter`
- Published as [`Edge-Center/edgecenter`](https://registry.terraform.io/providers/Edge-Center/edgecenter/latest) in the Terraform Registry
- Licence MPL-2.0, default branch `master`
- Actively developed

It imports every SDK below, which makes it the integration point of the whole
space. `edgecenter/config.go` is where the clients are built.

## Go SDKs

Standalone modules, one per service. The import path is the repository path.

| Module | Covers | Licence | Branch | Activity |
| :--- | :--- | :--- | :--- | :--- |
| [edgecentercloud-go](https://github.com/Edge-Center/edgecentercloud-go) `/v2` | Instances, networks, volumes, load balancers, Kubernetes, DBaaS, reseller | MPL-2.0 | `master` | Active |
| [edgecentercdn-go](https://github.com/Edge-Center/edgecentercdn-go) | CDN resources, origin groups, rules, certificates, ACME, shielding, statistics | - | `main` | Active |
| [edgecenterprotection-go](https://github.com/Edge-Center/edgecenterprotection-go) | DDoS Protection resources, aliases, certificates, black and white lists | MPL-2.0 | `master` | Active |
| [edgecenteredgemon-go](https://github.com/Edge-Center/edgecenteredgemon-go) | RMON checks, check groups, notification channels, status pages | - | `main` | Active |
| [edgecenter-dns-sdk-go](https://github.com/Edge-Center/edgecenter-dns-sdk-go) | DNS zones, RRSets, secondary zones | Apache-2.0 | `main` | Stable, Go 1.19 |
| [edgecenter-storage-sdk-go](https://github.com/Edge-Center/edgecenter-storage-sdk-go) | Object Storage buckets and credentials | MPL-2.0 | `main` | Stable, Go 1.19 |

Two shapes exist in this list, and knowing which you are in saves time:

**Transport core plus resource packages.** `edgecenteredgemon-go` is the reference
implementation and `edgecentercdn-go` mirrors it. An `edgecenter` package holds a
`Requester` interface, a `provider` sub-package with the HTTP client and request
signing, and typed errors with sentinels (`ErrNotFound`, `ErrConflict`,
`ErrRateLimit`). Each API resource gets its own sub-package. Unit tests use a mock
`Requester` from `internal/testutil`.

**Flat single package.** `edgecentercloud-go` keeps one `xxx.go` and `xxx_test.go`
pair per resource in a single top-level package, with a `Client` struct that
aggregates about thirty typed service fields. Older, but it covers the largest API
surface by a wide margin.

`edgecenter-dns-sdk-go` and `edgecenter-storage-sdk-go` predate both patterns and
still target Go 1.19.

## Kubernetes and platform

| Repository | What it does | Activity |
| :--- | :--- | :--- |
| [edgecenter-ccm](https://github.com/Edge-Center/edgecenter-ccm) | Cloud Controller Manager: node lifecycle, node addresses, `type: LoadBalancer` services. Ships as a container image, module name `ec-ccm`. | Active |
| [edgecenter-csi](https://github.com/Edge-Center/edgecenter-csi) | CSI driver for EdgeCenter Disks. Driver name `csi.edgecenter.org`, access mode `ReadWriteOnce`, Kubernetes 1.31 or newer, privileged containers required. | Active |
| [ec-helm-charts.github.io](https://github.com/Edge-Center/ec-helm-charts.github.io) | The Helm chart repository for everything above. Add it with `helm repo add edgecenter https://edge-center.github.io/ec-helm-charts.github.io/`. There is no browsable web page, only the chart index. | Active |
| [ec-plugin](https://github.com/Edge-Center/ec-plugin) | MKaaS plugin. | Active |
| [external-dns-ec-webhook](https://github.com/Edge-Center/external-dns-ec-webhook) | ExternalDNS webhook provider, so Kubernetes ingresses and services get EdgeCenter DNS records. Built on `edgecenter-dns-sdk-go`. | Active |
| [cert-manager-webhook-edgecenter](https://github.com/Edge-Center/cert-manager-webhook-edgecenter) | cert-manager DNS01 solver for EdgeCenter DNS. A fork carrying EdgeCenter support; module path uses the lowercase `edge-center` spelling. | Active |

## Integrations and tools

| Repository | What it does | Activity |
| :--- | :--- | :--- |
| [ec-dns-certbot-plugin](https://github.com/Edge-Center/ec-dns-certbot-plugin) | Certbot DNS01 authenticator for EdgeCenter DNS. Python, Apache-2.0. | Quiet since 2025 |
| [grafana-cdn-datasource](https://github.com/Edge-Center/grafana-cdn-datasource) | Grafana data source for CDN metrics. TypeScript, Apache-2.0. | Quiet since 2025 |
| [cdn-stats-datasource-plugin](https://github.com/Edge-Center/cdn-stats-datasource-plugin) | Grafana data source for CDN statistics, the earlier plugin. Apache-2.0, default branch `master`. | Quiet since 2024 |
| [edgecentercloud-git-hooks](https://github.com/Edge-Center/edgecentercloud-git-hooks) | Shared git hooks used across EdgeCenter repositories. Python. | Quiet since 2023 |
| [pylint-blocking-calls](https://github.com/Edge-Center/pylint-blocking-calls) | Pylint plugin that flags blocking calls inside async Python. Generic, not EdgeCenter specific, MIT. | Quiet since 2023 |

The two Grafana data sources overlap. `grafana-cdn-datasource` is the newer of the
two; treat `cdn-stats-datasource-plugin` as its predecessor.

## Examples and demos

Reference integrations of the EdgeCenter streaming platform into mobile
applications. All MIT, all last touched in 2023, kept as reading material rather
than as maintained samples.

| Repository | Platform |
| :--- | :--- |
| [ios_streams_demo](https://github.com/Edge-Center/ios_streams_demo) | iOS, live streaming |
| [ios_vod_demo](https://github.com/Edge-Center/ios_vod_demo) | iOS, video on demand |
| [android_streams_demo](https://github.com/Edge-Center/android_streams_demo) | Android, live streaming |
| [android_vod_demo](https://github.com/Edge-Center/android_vod_demo) | Android, video on demand |

## Reading the activity column

"Active" means the repository received changes this year and is on the release
path of a supported product. "Quiet" means the code works and is published, but no
one is adding to it - open an issue before building something that depends on it
changing.

Licences differ per repository: MPL-2.0 for the provider and most SDKs, Apache-2.0
for the DNS SDK and the plugins, MIT for the demos, and a few repositories carry
no licence file yet. Check `LICENSE` in the repository before you vendor anything.
