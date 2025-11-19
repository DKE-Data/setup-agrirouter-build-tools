# Setup Development Environment Action

This GitHub Action installs all the development tools required for the agrirouter project. It's designed to replace the custom Docker-based GitHub runners by providing the same toolset through a reusable composite action.

## Tools Installed

- **Go** (configurable version, default: 1.24.6)
- **Java 17** (OpenJDK)
- **Skaffold** (latest)
- **Kustomize** (latest)
- **Helm** (configurable version, default: v3.14.2)
- **Protocol Buffers (protoc)** (configurable version, default: 21.6)
- **System tools**: curl, unzip, jq, openssl

### Go Tools
- **go-test-coverage** (configurable version, default: v2@latest)
- **oapi-codegen** (configurable version, default: v1.15.0)
- **protoc-gen-go** (configurable version, default: v1.28.1)
- **lichen** (configurable version, default: v0.1.7)

## Usage

### Basic Usage

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: dke-data/agrirouter-github-runner@main
  - name: Run your build commands
    run: |
      go version
      skaffold version
      # ... your build steps
```

### With Custom Versions

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: dke-data/agrirouter-github-runner@main
    with:
      go-version: '1.23.0'
      helm-version: 'v3.13.0'
      protoc-version: '21.5'
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

## Architecture Support

This action automatically detects the runner architecture and installs the appropriate binaries for:
- `x86_64` (amd64)
- `aarch64` / `arm64`

## Migration from Custom Runners

If you're migrating from the custom Docker-based runners to blacksmith.io or standard GitHub runners, simply replace your runner configuration and add this action as the first step in your workflows.

### Before (Custom Runner)
```yaml
runs-on: self-hosted
```

### After (with this action)
```yaml
runs-on: ubuntu-latest  # or blacksmith runners
steps:
  - uses: actions/checkout@v4
  - uses: dke-data/agrirouter-github-runner@main
  # ... rest of your workflow
```

## Development

This action is based on the original Dockerfile used for custom GitHub runners. The tool versions are kept in sync with the Docker image to ensure compatibility.

## License

This action is provided under the same license as the agrirouter project.