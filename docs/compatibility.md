# Compatibility

Which versions work together, and which Go toolchain each module needs.

## Terraform provider

| | |
| :--- | :--- |
| Latest release | `v0.14.7` |
| Registry | [`Edge-Center/edgecenter`](https://registry.terraform.io/providers/Edge-Center/edgecenter/latest) |
| Terraform | 0.13 or newer |
| Go toolchain to build from source | 1.26 |

Pin it the usual way and let patch releases through:

```hcl
edgecenter = {
  source  = "Edge-Center/edgecenter"
  version = "~> 0.14"
}
```

The provider is pre-1.0. Minor bumps can change resource behavior, so read the
release notes before moving from `0.13` to `0.14`.

## SDK versions inside a provider release

The provider does not track SDK branches. It pins tagged releases in `go.mod`, so
these are the exact versions that shipped in `v0.14.7`:

| Module | Pinned in provider v0.14.7 |
| :--- | :--- |
| `edgecentercloud-go/v2` | `v2.6.7` |
| `edgecentercloud-go` (v1, legacy paths) | `v0.1.11` |
| `edgecentercdn-go` | `v0.2.7` |
| `edgecenterprotection-go` | `v0.1.7` |
| `edgecenter-dns-sdk-go` | `v0.2.4` |
| `edgecenter-storage-sdk-go` | `v0.2.1` |
| `edgecenteredgemon-go` | `v0.1.0` |

Both major versions of the cloud SDK are present at once: `v2` carries the
current code, and the unversioned v1 path is still imported by a few resources
that have not been migrated.

`edgecentercdn-go v0.2.6` and `v0.2.7` are tagged on a branch that has not been
merged into `main`, and `main` itself sits just past `v0.2.5`. The checked-out
branch therefore looks as if features are missing. Compare against the tag, not
the branch tip, when you need to know what the provider actually builds with:

```sh
git show v0.2.7:some/file.go
git diff v0.2.7 main -- some/file.go
```

## Go toolchain per module

The `go` directive is the minimum; anything newer works.

| Module | Minimum Go | CI runs on |
| :--- | :--- | :--- |
| `terraform-provider-edgecenter` | 1.26 | 1.26 |
| `edgecentercdn-go` | 1.26 | 1.26 |
| `edgecenteredgemon-go` | 1.26 | 1.26 |
| `edgecenterprotection-go` | 1.24 | 1.26 |
| `edgecentercloud-go/v2` | 1.22 | 1.22 |
| `edgecenter-dns-sdk-go` | 1.19 | - |
| `edgecenter-storage-sdk-go` | 1.19 | - |

If you consume several SDKs in one program, the highest minimum wins, so a build
that includes `edgecentercdn-go` needs Go 1.26.

## Kubernetes components

| Component | Latest release | Requirement |
| :--- | :--- | :--- |
| `edgecenter-csi` | `v1.1.2` | Kubernetes 1.31 or newer, privileged containers allowed |
| `edgecenter-ccm` | `v2.1.2` | Deployed as a linux/amd64 container image |
| `external-dns-ec-webhook` | `v1.0.0` | ExternalDNS with webhook provider support |
| `cert-manager-webhook-edgecenter` | `v0.2.0` | cert-manager with a DNS01 solver |
| `ec-plugin` | `v2.1.5` | MKaaS |

Charts for these come from the organization Helm repository:

```sh
helm repo add edgecenter https://edge-center.github.io/ec-helm-charts.github.io/
```

## Keeping this page true

Every number here is checkable in about a minute:

```sh
gh api /repos/Edge-Center/terraform-provider-edgecenter/releases/latest --jq .tag_name
gh api "/repos/Edge-Center/terraform-provider-edgecenter/contents/go.mod?ref=v0.14.7" \
  --jq .content | base64 -d | grep Edge-Center
```

Update it in the same pull request that bumps a pin. A stale compatibility table
is worse than no table, because people stop checking.
