# Setup Development Environment Action

This GitHub Action installs the development tools required for the agrirouter
project. It replaces the custom Docker-based GitHub runners by providing the
same toolset through a reusable composite action.

## Tools Installed

- **Go** (configurable, default: 1.24.6) — module/build caching is deliberately disabled
- **Java 17** (Temurin)
- **Skaffold** (latest)
- **Kustomize** (latest)
- **Helm** (configurable, default: v3.14.2)
- **Protocol Buffers (protoc)** (configurable, default: 21.6)

`curl`, `unzip`, `jq` and `openssl` are **not** installed by this action; they
are already present on the GitHub-hosted and blacksmith runner images.

### Go Tools

- **go-test-coverage** (configurable, default: v2@latest)
- **oapi-codegen** (configurable, default: v1.15.0)
- **protoc-gen-go** (configurable, default: v1.28.1)
- **lichen** (configurable, default: v0.1.7)

## Usage

### Basic Usage

```yaml
steps:
  - uses: actions/checkout@v7
  - uses: dke-data/setup-agrirouter-build-tools@main
  - name: Run your build commands
    run: |
      go version
      skaffold version
      # ... your build steps
```

### With Custom Versions

```yaml
steps:
  - uses: actions/checkout@v7
  - uses: dke-data/setup-agrirouter-build-tools@main
    with:
      go-version: '1.25.7'
      helm-version: 'v3.14.2'
      protoc-version: '21.6'
  - name: Run your build commands
    run: |
      # ... your build steps
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `go-version` | Version of Go to install | No | `1.24.6` |
| `helm-version` | Version of Helm to install | No | `v3.14.2` |
| `protoc-version` | Version of protoc to install | No | `21.6` |
| `protoc-gen-go-version` | Version of protoc-gen-go to install | No | `v1.28.1` |
| `oapi-codegen-version` | Version of oapi-codegen to install | No | `v1.15.0` |
| `go-test-coverage-version` | Version of go-test-coverage to install | No | `v2@latest` |
| `lichen-version` | Version of lichen to install | No | `v0.1.7` |

## Outputs

| Output | Description |
|--------|-------------|
| `go-version` | The version of Go that was installed |

## Go toolchain selection

`actions/setup-go` v6 and newer export `GOTOOLCHAIN=local`, which makes `go`
refuse to build a module whose `go.mod` requires a newer release than the
`go-version` installed here. Consumers of this action pin a range of `go` and
`toolchain` directives, several of them newer than the default above, so this
action resets `GOTOOLCHAIN=auto` immediately after the setup step. Go therefore
fetches whatever toolchain a repository asks for, which is how this action
behaved before the upgrade.

If you want the stricter behaviour in a specific workflow, set
`GOTOOLCHAIN: local` in that job's `env:` after calling this action.

## oapi-codegen stays on the v1 line

The `oapi-codegen` default deliberately tracks `github.com/deepmap/oapi-codegen`
(v1), not `github.com/oapi-codegen/oapi-codegen/v2`.

v2 rewrites external `$ref`s under `components.securitySchemes` into
self-referencing pointers — for example `openIdDev` becomes
`{"$ref": "#/components/securitySchemes/openIdDev"}` — which drops the actual
`openIdConnectUrl` definitions from the embedded spec. Every agrirouter service
declares its security schemes through external `$ref`s, so with v2 the request
validator can no longer resolve them and rejects every authenticated request
with `security scheme "..." is not declared` (HTTP 403 instead of 401).

Adopting v2 requires either restructuring `securitySchemes` in
`agrirouter-api-specs` or an upstream fix. Until then, keep this on v1.

## Architecture Support

The action detects the runner architecture and installs matching binaries for:

- `x86_64` (amd64)
- `aarch64` / `arm64`

## Runner requirements

The actions used here (`actions/setup-go` v7, `actions/setup-java` v5) run on
Node 24 and require GitHub Actions runner **v2.327.1** or newer.

## Migration from Custom Runners

Replace the runner configuration and add this action as the first step after
checkout.

### Before (Custom Runner)

```yaml
runs-on: self-hosted
```

### After (with this action)

```yaml
runs-on: ubuntu-latest  # or blacksmith runners
steps:
  - uses: actions/checkout@v7
  - uses: dke-data/setup-agrirouter-build-tools@main
  # ... rest of your workflow
```

## License

This action is provided under the same license as the agrirouter project.
