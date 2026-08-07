# Development

What you need to know before changing code in this space. The organization-wide
rules on branching, signing and review are in
[CONTRIBUTING](https://github.com/Edge-Center/.github/blob/main/CONTRIBUTING.md);
this page is about the repositories themselves.

## There is no monorepo

Each repository is a separate Go module with its own `Makefile`, its own workflows
and its own release cadence. Nothing builds from the top. Work inside one
repository at a time.

## Make targets

The SDK repositories share one set:

```sh
make build         # go build ./...
make test          # go test -race ./...
make lint          # golangci-lint run ./...
make fmt           # gofmt plus goimports with the module as -local
make cover-check   # coverage gate
make tidy          # go mod tidy
```

The Terraform provider deviates:

```sh
make build         # builds into the local plugin path for dev_overrides
make linters       # golangci-lint, note the plural
make docs          # regenerate docs/ with tfplugindocs
go test ./...      # unit tests, there is no blanket make test target
```

`edgecentercloud-go` uses `make check-coverage` rather than `make cover-check`, and
`edgecentercdn-go` adds `make build-cli` for the `edge-cli` binary.

Run a single test the usual way:

```sh
go test -race -run TestName ./path/to/pkg
go test -race -run 'TestName/subtest' ./path/to/pkg
```

## Layout of the SDKs

Two patterns, described in the [repository map](repository-map.md). When you add
code, mirror the files already in the package instead of introducing a third
pattern.

`edgecenteredgemon-go` is the reference implementation. If you need to know how a
new SDK should look - transport core, typed errors, one package per resource,
mock `Requester` for tests, in-process integration harness - read that repository
first.

## Tests

**Unit tests** run with no network. The transport-core SDKs mock the `Requester`
interface (`internal/testutil`); `edgecentercloud-go` uses `httptest` servers.
These are what CI enforces, and they are what a pull request needs.

**Integration tests** in `edgecenteredgemon-go` wire the real client to an
in-process fake server and drive full CRUD without credentials. They run as part
of `make test`. This is the pattern to copy: it catches wiring mistakes that mocks
cannot, and it costs nothing to run.

**Live tests** need real credentials and are behind build tags. `make
test-integration` with `-tags=integration` and a `TestLive` filter, driven by
environment variables such as `EDGECENTER_RMON_BASE_URL` and
`EDGECENTER_RMON_API_KEY`.

**Acceptance tests** in the Terraform provider create real cloud resources against
the real API and cost real money. They are split by build tag - `test_cloud_resource`,
`test_cloud_data_source`, `test_not_cloud`, `test_cdn`, `test_dns`, `test_storage`,
`test_mkaas_*`, `test_sweep` - and require an environment file. Do not run them
casually, and run `test_sweep` afterwards to clean up what leaked.

## Working on the provider locally

Build the provider and point Terraform at your binary with a `dev_overrides` block
in `~/.terraformrc`:

```hcl
provider_installation {
  dev_overrides {
    "Edge-Center/edgecenter" = "/absolute/path/to/terraform-provider-edgecenter/bin"
  }
  direct {}
}
```

```sh
make build
```

With `dev_overrides` active, **do not run `terraform init`**. Terraform would go to
the registry and fail; overrides are consulted at plan and apply time instead.
Terraform prints a warning on every command telling you overrides are in effect -
that warning is the confirmation your build is being used.

### Verify against the API, not only against the plan

A clean plan is not proof. The provider can report success while the change never
reached the API, so check the result with a direct request as well:

```sh
curl -H "Authorization: APIKey $EC_PERMANENT_TOKEN" \
  https://api.edgecenter.online/cloud/v1/networks/1/1
```

Isolate what you are testing, too. An unrelated defect in the same resource will
mask the one you are looking at.

## Style

- `make fmt` before committing. CI fails on unformatted code.
- Names carry the meaning. Comment only where the *why* is genuinely not obvious,
  and never to restate the next line.
- Wrap errors with context: `fmt.Errorf("create zone %q: %w", name, err)`.
- Follow the file next to yours. A new pattern in an old package costs more than
  it is worth.

## API paths that do not match resource names

Useful when you are reading the SDKs against the API reference:

- **CDN.** Rules live under `/cdn/resources/{id}/locations`, origin groups under
  `/cdn/source_groups`, certificates under `/cdn/ssl/certificates`, ACME under
  `/cdn/resources/{id}/ssl/le/*`.
- **DNS.** Everything under `/dns/v2`: `/zones`, `/zones/{zone}`, an RRSet at
  `/zones/{zone}/{name}/{type}`, and `/secondary-zones/{zone}`.
- **Cloud.** Base path plus `/cloud`, then version, resource, project and region:
  `/cloud/v1/lblisteners/{project}/{region}`.

The authorization header is `Authorization: APIKey <token>` everywhere.
