# Anchor CLI Build Actions

This repository contains GitHub Actions workflows for building and setting up the Anchor CLI with specific versions or commit hashes. It solves the common problem of needing to use a particular version of Anchor CLI in CI/CD pipelines, especially when working with specific commits or versions that aren't officially released.

## Motivation

- Need to build Anchor CLI from specific commits for testing or compatibility reasons
- Support for multiple architectures (x86_64 and aarch64)
- Cached builds to improve CI/CD performance
- Avoid rate limiting issues when downloading Anchor CLI in GitHub Actions

## Workflows

### 1. Build Anchor CLI (`anchor-cli.yml`)

This workflow builds the Anchor CLI from source and publishes it as a release.

#### Features:
- Builds from specific commit hash or version tag
- Supports multiple architectures (x86_64-unknown-linux-musl, aarch64-unknown-linux-musl)
- Uses sccache for faster builds
- Caches built binaries for reuse
- Automatically creates GitHub releases with built binaries

#### Usage:

You can trigger this workflow manually with these parameters:

```yaml
inputs:
  rust_version: 
    default: 'stable'
  rev:  # Specific commit hash (optional)
  anchor_version:
    default: '0.30.1'
  target:
    default: 'x86_64-unknown-linux-musl'
    options:
      - x86_64-unknown-linux-musl
      - aarch64-unknown-linux-musl
```

### 2. Download and Setup Action (`action.yml`)

This composite action downloads and sets up the pre-built Anchor CLI in your workflow.

#### Features:
- Downloads pre-built binaries from releases
- Handles GitHub rate limiting using GITHUB_TOKEN
- Supports version or commit-specific installations
- Adds Anchor CLI to PATH automatically

#### Usage:

```yaml
- uses: your-org/your-repo@main
  with:
    # Use either rev or version
    rev: 'specific-commit-hash'  # optional
    version: '0.30.1'           # optional, defaults to 0.30.1
    target: 'x86_64-unknown-linux-musl'  # optional, defaults to x86_64-unknown-linux-musl
```

## Example Workflow

Here's an example of how to use both actions together:

```yaml
name: Test with specific Anchor version

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      # First, build the specific version you need
      - name: Build Anchor CLI
        uses: ./.github/workflows/anchor-cli.yml
        with:
          rev: 'your-specific-commit'
          target: 'x86_64-unknown-linux-musl'

      # Then use it in your workflow
      - name: Setup Anchor CLI
        uses: ./.github/actions/download-anchor-cli
        with:
          rev: 'your-specific-commit'
          target: 'x86_64-unknown-linux-musl'

      - name: Verify installation
        run: anchor --version
```

## Requirements

- GitHub Actions runner with Linux
- Appropriate permissions for creating releases
- AWS credentials if using sccache with S3

## Contributing

Feel free to open issues and pull requests for improvements and bug fixes.