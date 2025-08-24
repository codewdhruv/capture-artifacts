# Capture Artifact Information

A GitHub Action that captures comprehensive information about build artifacts, including metadata about the build environment, repository state, and artifact details.

## Features

- ✅ Captures artifact name, location, and tag
- 📊 Includes build metadata (commit SHA, timestamp, workflow info)
- 📁 Supports multiple output formats (JSON, YAML, ENV)
- 🔍 Validates artifact existence and captures file size
- 🔄 Reusable across any workflow
- 📤 Provides outputs for use in subsequent steps

## Usage

### Basic Usage

```yaml
- name: Capture artifact information
  uses: codewdhruv/capture-artifacts@v1
  with:
    artifact-name: 'my-app'
    artifact-location: './dist/app.tar.gz'
    artifact-tag: 'v1.2.3'
```

### Advanced Usage

```yaml
- name: Capture artifact information
  id: artifact-capture
  uses: codewdhruv/capture-artifacts@v1
  with:
    artifact-name: 'my-application'
    artifact-location: './build/output.zip'
    artifact-tag: 'v${{ github.run_number }}'
    output-format: 'json'
    output-file: 'artifact-metadata'

- name: Use captured information
  run: |
    echo "Artifact: ${{ fromJson(steps.artifact-capture.outputs.artifact-info).artifact.name }}"
    echo "Built at: ${{ fromJson(steps.artifact-capture.outputs.artifact-info).build.timestamp }}"
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `artifact-name` | Name of the artifact | ✅ Yes | - |
| `artifact-location` | Location/path of the artifact | ✅ Yes | - |
| `artifact-tag` | Tag or version of the artifact | ❌ No | `latest` |
| `output-format` | Output format (`json`, `yaml`, `env`) | ❌ No | `json` |
| `output-file` | Output file name (without extension) | ❌ No | `artifact-info` |

## Outputs

| Name | Description |
|------|-------------|
| `artifact-info` | Complete artifact information in JSON format |
| `output-file-path` | Path to the generated output file |

## Output Structure

### JSON Format

```json
{
  "artifact": {
    "name": "my-app",
    "location": "./dist/app.tar.gz",
    "tag": "v1.2.3",
    "exists": true,
    "size": "1048576"
  },
  "build": {
    "timestamp": "2024-01-15T10:30:45Z",
    "repository": "owner/repo-name",
    "repository_url": "https://github.com/owner/repo-name",
    "commit_sha": "abc123def456...",
    "commit_short_sha": "abc123d",
    "ref": "refs/heads/main",
    "workflow_name": "Build and Deploy",
    "run_id": "123456789",
    "run_number": "42"
  }
}
```

### YAML Format

```yaml
artifact:
  name: my-app
  location: ./dist/app.tar.gz
  tag: v1.2.3
  exists: true
  size: "1048576"
build:
  timestamp: "2024-01-15T10:30:45Z"
  repository: owner/repo-name
  repository_url: https://github.com/owner/repo-name
  commit_sha: abc123def456...
  commit_short_sha: abc123d
  ref: refs/heads/main
  workflow_name: Build and Deploy
  run_id: "123456789"
  run_number: "42"
```

### Environment Variables Format

```bash
ARTIFACT_NAME=my-app
ARTIFACT_LOCATION=./dist/app.tar.gz
ARTIFACT_TAG=v1.2.3
ARTIFACT_EXISTS=true
ARTIFACT_SIZE=1048576
BUILD_TIMESTAMP=2024-01-15T10:30:45Z
BUILD_REPOSITORY=owner/repo-name
BUILD_REPOSITORY_URL=https://github.com/owner/repo-name
BUILD_COMMIT_SHA=abc123def456...
BUILD_COMMIT_SHORT_SHA=abc123d
BUILD_REF=refs/heads/main
BUILD_WORKFLOW_NAME=Build and Deploy
BUILD_RUN_ID=123456789
BUILD_RUN_NUMBER=42
```

## Common Use Cases

### 1. Docker Image Information

```yaml
- name: Build Docker image
  run: docker build -t myapp:${{ github.sha }} .

- name: Capture Docker image info
  uses: codewdhruv/capture-artifacts@v1
  with:
    artifact-name: 'myapp-docker'
    artifact-location: 'myapp:${{ github.sha }}'
    artifact-tag: '${{ github.sha }}'
```

### 2. Multi-Platform Builds

```yaml
strategy:
  matrix:
    platform: [linux, windows, macos]

steps:
  - name: Build for ${{ matrix.platform }}
    run: ./build.sh ${{ matrix.platform }}
    
  - name: Capture artifact info
    uses: codewdhruv/capture-artifacts@v1
    with:
      artifact-name: 'myapp-${{ matrix.platform }}'
      artifact-location: './dist/myapp-${{ matrix.platform }}'
      artifact-tag: 'v1.0.0-${{ matrix.platform }}'
```

### 3. Integration with Artifact Upload

```yaml
- name: Capture artifact info
  id: capture
  uses: codewdhruv/capture-artifacts@v1
  with:
    artifact-name: 'build-output'
    artifact-location: './dist'
    artifact-tag: 'build-${{ github.run_number }}'

- name: Upload artifacts with metadata
  uses: actions/upload-artifact@v4
  with:
    name: ${{ fromJson(steps.capture.outputs.artifact-info).artifact.name }}
    path: |
      ${{ fromJson(steps.capture.outputs.artifact-info).artifact.location }}
      ${{ steps.capture.outputs.output-file-path }}
```

## Requirements

- GitHub Actions runner with bash support
- `jq` (pre-installed on GitHub-hosted runners)
- `stat` command (available on most Unix systems)

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This project is licensed under the MIT License - see the LICENSE file for details.
