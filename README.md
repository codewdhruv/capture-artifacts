# 📦 Capture Artifact Information Action

[![GitHub release](https://img.shields.io/github/release/codewdhruv/capture-artifacts.svg)](https://github.com/codewdhruv/capture-artifacts/releases)
[![GitHub marketplace](https://img.shields.io/badge/marketplace-capture--artifact--info-blue?logo=github)](https://github.com/marketplace/actions/capture-artifact-information)

A GitHub Action that captures comprehensive information about build artifacts, including metadata about the build environment, repository state, and artifact details. Perfect for CI/CD workflows that need to track and document their build outputs.

## Features

- **Comprehensive metadata**: Captures artifact name, location, tag, size, and existence
- **Build context**: Includes repository info, commit details, and workflow metadata
- **Multiple formats**: Supports JSON, YAML, and environment variable outputs
- **Easy integration**: Works seamlessly with existing GitHub workflows
- **Documentation**: Generates structured metadata for compliance and auditing

## Quick start

```yaml
- name: Capture artifact information
  uses: codewdhruv/capture-artifacts@v1
  with:
    artifact-name: 'my-application'
    artifact-location: './dist/app.tar.gz'
    artifact-tag: 'v1.2.3'
```

## Usage examples

### Basic usage

```yaml
name: Build and Track Artifacts

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Your build steps here
      - name: Build application
        run: npm run build && tar -czf dist.tar.gz dist/
      
      # Capture artifact metadata
      - name: Capture artifact info
        id: artifact
        uses: codewdhruv/capture-artifacts@v1
        with:
          artifact-name: 'web-app'
          artifact-location: './dist.tar.gz'
          artifact-tag: 'v${{ github.run_number }}'
          output-format: 'json'
      
      # Upload with metadata
      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-artifacts
          path: |
            dist.tar.gz
            ${{ steps.artifact.outputs.output-file-path }}
```

### Docker image tracking

```yaml
- name: Build Docker image
  run: |
    docker build -t myapp:${{ github.sha }} .
    docker save myapp:${{ github.sha }} -o myapp.tar

- name: Track Docker artifact
  uses: codewdhruv/capture-artifacts@v1
  with:
    artifact-name: 'myapp-docker'
    artifact-location: './myapp.tar'
    artifact-tag: '${{ github.sha }}'
```

### Multi-platform builds

```yaml
strategy:
  matrix:
    platform: [linux-amd64, linux-arm64, windows-amd64]

steps:
  - name: Build for ${{ matrix.platform }}
    run: ./build.sh ${{ matrix.platform }}
    
  - name: Track platform artifact
    uses: codewdhruv/capture-artifacts@v1
    with:
      artifact-name: 'myapp-${{ matrix.platform }}'
      artifact-location: './dist/myapp-${{ matrix.platform }}'
      artifact-tag: 'v1.0.0-${{ matrix.platform }}'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `artifact-name` | Name of the artifact | ✅ Yes | - |
| `artifact-location` | Path to artifact (file/directory) | ✅ Yes | - |
| `artifact-tag` | Version tag for the artifact | ❌ No | `latest` |
| `output-format` | Output format (`json`/`yaml`/`env`) | ❌ No | `json` |
| `output-file` | Output filename (no extension) | ❌ No | `artifact-info` |

## Outputs

| Output | Description |
|--------|-------------|
| `artifact-info` | Complete artifact information in JSON |
| `output-file-path` | Path to the generated metadata file |

## Output Structure

<details>
<summary>📋 JSON Format Example</summary>

```json
{
  "artifact": {
    "name": "web-app",
    "location": "./dist.tar.gz",
    "tag": "v1.2.3",
    "exists": true,
    "size": "2048576"
  },
  "build": {
    "timestamp": "2024-01-15T10:30:45Z",
    "repository": "owner/repo",
    "repository_url": "https://github.com/owner/repo",
    "commit_sha": "abc123def456...",
    "commit_short_sha": "abc123d",
    "ref": "refs/heads/main",
    "workflow_name": "CI/CD",
    "run_id": "123456789",
    "run_number": "42"
  }
}
```
</details>

## Advanced Use Cases

### Integration with deployment

```yaml
- name: Capture artifact info
  id: capture
  uses: codewdhruv/capture-artifacts@v1
  with:
    artifact-name: 'production-build'
    artifact-location: './build'
    artifact-tag: 'release-${{ github.ref_name }}'

- name: Deploy to production
  env:
    ARTIFACT_NAME: ${{ fromJson(steps.capture.outputs.artifact-info).artifact.name }}
    BUILD_SHA: ${{ fromJson(steps.capture.outputs.artifact-info).build.commit_short_sha }}
  run: |
    echo "Deploying $ARTIFACT_NAME built from $BUILD_SHA"
    # Your deployment logic here
```

### Compliance documentation

```yaml
- name: Generate compliance report
  uses: codewdhruv/capture-artifacts@v1
  with:
    artifact-name: 'compliance-build'
    artifact-location: './release'
    artifact-tag: 'audit-${{ github.run_number }}'
    output-format: 'yaml'
    output-file: 'compliance-report'

- name: Archive compliance data
  uses: actions/upload-artifact@v4
  with:
    name: compliance-documentation
    path: compliance-report.yaml
    retention-days: 365
```

## 🤝 Contributing

Contributions are welcome! Please read our [Contributing Guidelines](CONTRIBUTING.md) and feel free to submit issues or pull requests.

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Support

If you find this action helpful, please:
- ⭐ Star this repository
- 🐛 Report issues on [GitHub Issues](https://github.com/codewdhruv/capture-artifacts/issues)
- 💬 Join discussions in [GitHub Discussions](https://github.com/codewdhruv/capture-artifacts/discussions)

---

**Made with ❤️ for the GitHub Actions community**
