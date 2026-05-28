# Cortex Cloud Scan GitHub Action

[![license](https://img.shields.io/badge/license-MIT-blue.svg)](./LICENSE) [![support](https://img.shields.io/badge/Support%20Level-Community-yellowgreen)](./SUPPORT.md)

A GitHub Action wrapper for the Cortex CLI that enables security scanning for Code Security, Container Workload Protection (CWP), and API Security.

## Features

- **Code Security**: Scan source code for secrets, IaC misconfigurations, and dependency vulnerabilities (SCA)
- **Container Workload Protection**: Scan container images for vulnerabilities and malware
- **API Security**: Test APIs for security vulnerabilities and misconfigurations

## Prerequisites

- Active Cortex Cloud license with appropriate module add-ons
- Cortex Cloud API credentials (API Key, API Key ID, and API Base URL)
- For API Security scans: Java 11 or higher must be available

## Quick Start

### Code Security Scan

```yaml
- name: Cortex Code Security Scan
  uses: PaloAltoNetworks/cortex-cloud-scan@v1
  with:
    scan-type: code
    api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
    api-key: ${{ secrets.CORTEX_API_KEY }}
    api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
    directory: .
    repo-id: ${{ github.repository }}
    branch: ${{ github.ref_name }}
    upload-mode: upload
```

### Container Image Scan

```yaml
- name: Cortex Container Scan
  uses: PaloAltoNetworks/cortex-cloud-scan@v1
  with:
    scan-type: image
    api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
    api-key: ${{ secrets.CORTEX_API_KEY }}
    api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
    image: myapp:latest
```

### API Security Scan

```yaml
- name: Cortex API Security Scan
  uses: PaloAltoNetworks/cortex-cloud-scan@v1
  with:
    scan-type: api
    api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
    api-key: ${{ secrets.CORTEX_API_KEY }}
    api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
    scanned-app-url: https://api.example.com
    api-spec-file: ./openapi.yaml
```

## Inputs

### Required for All Scans

| Input | Description |
|-------|-------------|
| `api-base-url` | Cortex Cloud API base URL |
| `api-key` | Cortex Cloud API key |
| `api-key-id` | Cortex Cloud API key ID |
| `scan-type` | Type of scan: `code`, `image`, or `api` |

### Code Security Options

| Input | Description | Default |
|-------|-------------|---------|
| `directory` | Directory path to scan | - |
| `file` | Single file path to scan | - |
| `repo-id` | Repository ID (format: owner/repo) | - |
| `branch` | Branch name | - |
| `upload-mode` | Upload mode: `upload`, `no-upload`, or `no-code` | `upload` |
| `framework` | Frameworks to scan (comma-separated) | - |
| `skip-framework` | Frameworks to skip (comma-separated) | - |
| `severity` | Filter by severity: `critical`, `high`, `medium`, `low`, `unknown` | - |
| `output-format` | Output format: `cli`, `json`, `spdx`, `junitxml`, `sarif`, `cyclonedx`, `cyclonedx_json` | `cli` |
| `output-file-path` | Path for output file | - |
| `soft-fail` | Do not fail pipeline on findings | `false` |
| `no-fail-on-crash` | Return exit code 0 on internal errors | `false` |
| `compact` | Do not display code blocks in output | `false` |
| `skip-path` | Path to skip during scan | - |
| `create-repo-if-missing` | Create repository if missing | `false` |
| `validate-secrets` | Validate detected secrets | `false` |

### Container Workload Protection Options

| Input | Description | Default |
|-------|-------------|---------|
| `image` | Container image name to scan | - |
| `archive` | Scan from archive file | `false` |
| `archive-format` | Archive format: `docker-archive` or `oci-archive` | `docker-archive` |
| `docker-host` | Path to Docker socket | - |
| `ci-pipeline-id` | CI pipeline identifier | - |
| `ci-build-id` | CI build identifier | - |
| `image-timeout` | Timeout in seconds | `60` |
| `image-output-format` | Output format: `human-readable` or `json` | `human-readable` |
| `soft-fail` | Do not fail pipeline on findings | `false` |
| `no-fail-on-crash` | Return exit code 0 on internal errors | `false` |

### API Security Options

| Input | Description | Default |
|-------|-------------|---------|
| `scanned-app-url` | Base URL of the application to scan | - |
| `api-spec-file` | Path to API specification file | - |
| `api-spec-type` | API specification type | `openapi` |
| `auth-file` | Path to authentication file | - |
| `java-location` | Path to Java binary (version >= 11) | `java` |
| `api-timeout` | Scan timeout in seconds | `300` |
| `concurrency` | Concurrency limit for scan requests | `5` |
| `api-output-file` | Output path for report file | - |

### Common Options

| Input | Description | Default |
|-------|-------------|---------|
| `ca-certificate` | Path to CA certificate for proxy/TLS inspection | - |
| `no-cert-verify` | Disable TLS certificate verification | `false` |
| `http-proxy` | HTTP proxy server URL | - |
| `log-level` | Logging level: `INFO`, `WARNING`, `ERROR` | `INFO` |
| `cortexcli-version` | Version of cortexcli to use | `latest` |

## Outputs

| Output | Description |
|--------|-------------|
| `scan-result` | Path to scan result file (if output file was specified) |
| `exit-code` | Exit code from the scan |

## Complete Examples

### Code Security: Full Repository Scan with SARIF Output

```yaml
name: Code Security Scan

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  code-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run Cortex Code Security Scan
        uses: PaloAltoNetworks/cortex-cloud-scan@v1
        with:
          scan-type: code
          api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
          api-key: ${{ secrets.CORTEX_API_KEY }}
          api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
          directory: .
          repo-id: ${{ github.repository }}
          branch: ${{ github.ref_name }}
          upload-mode: upload
          framework: terraform,kubernetes,secrets
          severity: critical,high
          output-format: sarif
          output-file-path: cortex-results.sarif

      - name: Upload SARIF results
        uses: github/codeql-action/upload-sarif@v2
        if: always()
        with:
          sarif_file: cortex-results.sarif
```

### Code Security: Scan Specific Frameworks Only

```yaml
- name: Scan Terraform and Secrets Only
  uses: PaloAltoNetworks/cortex-cloud-scan@v1
  with:
    scan-type: code
    api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
    api-key: ${{ secrets.CORTEX_API_KEY }}
    api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
    directory: ./infrastructure
    repo-id: ${{ github.repository }}
    branch: ${{ github.ref_name }}
    framework: terraform,secrets
    soft-fail: true
```

### Code Security: Local Scan Without Upload

```yaml
- name: Local Code Scan
  uses: PaloAltoNetworks/cortex-cloud-scan@v1
  with:
    scan-type: code
    api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
    api-key: ${{ secrets.CORTEX_API_KEY }}
    api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
    directory: .
    upload-mode: no-upload
    severity: critical,high
    output-format: json
    output-file-path: scan-results.json
```

### Container: Build and Scan Docker Image

```yaml
name: Build and Scan Container

on:
  push:
    branches: [ main ]

jobs:
  build-and-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .

      - name: Scan container image
        uses: PaloAltoNetworks/cortex-cloud-scan@v1
        with:
          scan-type: image
          api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
          api-key: ${{ secrets.CORTEX_API_KEY }}
          api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
          image: myapp:${{ github.sha }}
          ci-pipeline-id: ${{ github.run_id }}
          ci-build-id: ${{ github.run_number }}
          soft-fail: false
```

### Container: Scan from Archive

```yaml
- name: Scan container from archive
  uses: PaloAltoNetworks/cortex-cloud-scan@v1
  with:
    scan-type: image
    api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
    api-key: ${{ secrets.CORTEX_API_KEY }}
    api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
    image: ./myapp.tar
    archive: true
    archive-format: docker-archive
```

### API Security: Scan with Authentication

```yaml
- name: API Security Scan with Auth
  uses: PaloAltoNetworks/cortex-cloud-scan@v1
  with:
    scan-type: api
    api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
    api-key: ${{ secrets.CORTEX_API_KEY }}
    api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
    scanned-app-url: https://api.example.com
    api-spec-file: ./specs/openapi.yaml
    auth-file: ./auth-config.yaml
    api-timeout: 600
    concurrency: 10
    api-output-file: api-scan-results.json
```

### Multi-Module Pipeline

```yaml
name: Full Security Scan

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  code-security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Code Security Scan
        uses: PaloAltoNetworks/cortex-cloud-scan@v1
        with:
          scan-type: code
          api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
          api-key: ${{ secrets.CORTEX_API_KEY }}
          api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
          directory: .
          repo-id: ${{ github.repository }}
          branch: ${{ github.ref_name }}

  container-security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build image
        run: docker build -t myapp:latest .

      - name: Container Scan
        uses: PaloAltoNetworks/cortex-cloud-scan@v1
        with:
          scan-type: image
          api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
          api-key: ${{ secrets.CORTEX_API_KEY }}
          api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
          image: myapp:latest

  api-security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: API Security Scan
        uses: PaloAltoNetworks/cortex-cloud-scan@v1
        with:
          scan-type: api
          api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
          api-key: ${{ secrets.CORTEX_API_KEY }}
          api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
          scanned-app-url: https://api.example.com
          api-spec-file: ./openapi.yaml
```

### Using Proxy and Custom CA Certificate

```yaml
- name: Scan with Corporate Proxy
  uses: PaloAltoNetworks/cortex-cloud-scan@v1
  with:
    scan-type: code
    api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
    api-key: ${{ secrets.CORTEX_API_KEY }}
    api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
    directory: .
    repo-id: ${{ github.repository }}
    branch: ${{ github.ref_name }}
    http-proxy: http://proxy.company.com:8080
    ca-certificate: /path/to/company-ca.crt
```

## Security Best Practices

1. **Store Credentials Securely**: Always use GitHub Secrets for API credentials
2. **Limit Permissions**: Use API keys with minimum required permissions
3. **Review Results**: Set `soft-fail: false` to block deployments on findings
4. **Upload Results**: Use `upload-mode: upload` to track findings in Cortex Cloud
5. **Regular Scans**: Run scans on every push and pull request

## Troubleshooting

### Common Issues

**Issue**: "scan-type must be one of: code, image, api"
- **Solution**: Ensure the `scan-type` input is set to exactly `code`, `image`, or `api`

**Issue**: "For code scans, either 'directory' or 'file' input is required"
- **Solution**: Provide either `directory` or `file` input for code scans

**Issue**: "TLS certificate verification failed"
- **Solution**: Use the `ca-certificate` input to provide your CA certificate, or set `no-cert-verify: true` (not recommended for production)

**Issue**: API scan fails with Java error
- **Solution**: Ensure Java 11 or higher is installed. Use `actions/setup-java@v4` before the scan

## Version History

* 1.0.0
    * Initial Release
    * Support for Code Security, Container Workload Protection, and API Security scans
    * Comprehensive input options for all scan types

## License

This project is licensed under the MIT License - see the [LICENSE](./LICENSE) file for details

## Support

For issues and questions:
- Review the [Cortex CLI documentation](https://docs-cortex.paloaltonetworks.com/)
- Check existing [GitHub Issues](../../issues)
- Contact Palo Alto Networks support

## Contributing

See [CONTRIBUTING](./CONTRIBUTING.md) for details on how to contribute to this project.
