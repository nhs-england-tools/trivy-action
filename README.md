# Trivy Actions

This repository contains a collection of shared GitHub Actions that extend and augment the capabilities of [`aquasecurity/trivy-action`](https://github.com/aquasecurity/trivy-action). These actions provide streamlined workflows for security scanning in CI/CD pipelines.

## Usage

When referencing these actions in your workflows, **always use a specific release tag** rather than `@main` for production environments. This ensures stability and prevents unexpected breaking changes from affecting your CI/CD pipelines.

### Recommended Usage Pattern

```yaml
# ✅ Good - Use specific release tag
uses: nhs-england-tools/trivy-action/iac-scan@v1.0.0

# ❌ Avoid - Using main branch in production
uses: nhs-england-tools/trivy-action/iac-scan@main
```

### Finding Available Releases

Check the [Releases page](https://github.com/nhs-england-tools/trivy-action/releases) for the latest stable version and release notes.

## Available Actions

### 🏗️ Infrastructure as Code (IaC) Scan

**Path:** `./iac-scan`

Performs comprehensive Trivy Infrastructure as Code scanning and reporting for Terraform and other IaC files.

#### Usage

```yaml
- name: Run Trivy IaC Scan
  uses: nhs-england-tools/trivy-action/iac-scan@v1.0.0
  with:
    scan-ref: './terraform'
    severity: 'HIGH,CRITICAL'
    fail-on-critical-high: 'true'
```

#### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `scan-ref` | Directory or file to scan | No | `./terraform` |
| `severity` | Comma-separated list of severity levels | No | `HIGH,CRITICAL,MEDIUM,LOW,UNKNOWN` |
| `trivy-config` | Path to Trivy configuration file | No | `trivy.yaml` |
| `artifact-name` | Name for the uploaded artifact | No | `trivy-iac-scan-results` |
| `fail-on-critical-high` | Fail action on critical/high findings | No | `true` |

#### Outputs

| Output | Description |
|--------|-------------|
| `critical-count` | Number of critical severity findings |
| `high-count` | Number of high severity findings |
| `report-path` | Path to the generated markdown report |

### 📦 SBOM Scan

**Path:** `./sbom-scan`

Performs Software Bill of Materials (SBOM) scanning and reporting with optional GitHub Dependency Graph integration.

#### Usage

```yaml
- name: Generate SBOM
  uses: nhs-england-tools/trivy-action/sbom-scan@v1.0.0
  with:
    image-ref: 'myapp:latest'
    publish-to-dependency-graph: 'true'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

#### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `image-ref` | Docker image reference to scan | Yes | - |
| `github-token` | GitHub token for dependency graph upload | No | - |
| `publish-to-dependency-graph` | Publish SBOM to GitHub Dependency Graph | No | `false` |
| `artifact-name` | Name for the uploaded SBOM artifact | No | `sbom` |

#### Outputs

| Output | Description |
|--------|-------------|
| `sbom-path` | Path to the generated SBOM file |

## Complete Workflow Example

```yaml
name: Security Scanning

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  iac-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run IaC Security Scan
        uses: nhs-england-tools/trivy-action/iac-scan@v1.0.0
        with:
          scan-ref: './infrastructure'
          severity: 'HIGH,CRITICAL'
          fail-on-critical-high: 'true'

  sbom-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Docker Image
        run: docker build -t myapp:${{ github.sha }} .
      
      - name: Generate SBOM
        uses: nhs-england-tools/trivy-action/sbom-scan@v1.0.0
        with:
          image-ref: 'myapp:${{ github.sha }}'
          publish-to-dependency-graph: 'true'
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Features

- **IaC Security Scanning**: Comprehensive security analysis for Infrastructure as Code
- **SBOM Generation**: Software Bill of Materials creation in SPDX format
- **GitHub Integration**: Optional dependency graph publishing
- **Artifact Upload**: Automatic upload of scan results and SBOMs
- **Configurable Severity**: Customizable severity level filtering
- **Flexible Configuration**: Support for custom Trivy configuration files

## Requirements

- GitHub Actions environment
- Docker images (for SBOM scanning)
- Terraform/IaC files (for infrastructure scanning)

**Related Projects:**
- [Trivy](https://github.com/aquasecurity/trivy) - Main security scanner
- [Trivy Action](https://github.com/aquasecurity/trivy-action) - Base GitHub Action