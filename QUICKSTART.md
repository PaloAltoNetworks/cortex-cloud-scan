# Quick Start Guide

This guide will help you get started with the Cortex Cloud Scan GitHub Action in 5 minutes.

## Prerequisites

Before you begin, ensure you have:

1. **Cortex Cloud Account**: An active Cortex Cloud tenant with appropriate licenses
2. **API Credentials**: Generated from your Cortex Cloud console
   - API Base URL
   - API Key
   - API Key ID

## Step 1: Generate API Credentials

1. Log in to your Cortex Cloud console
2. Navigate to **Settings** → **Configurations** → **API Keys**
3. Click **Generate API key**
4. Select the appropriate permissions:
   - For read-only scans: **CLI Tools View**
   - For uploading results: **CLI Tools View/Edit**
5. Save your credentials:
   - API Base URL
   - API Key ID
   - API Key

## Step 2: Add Secrets to GitHub

1. Go to your GitHub repository
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret** and add:
   - Name: `CORTEX_API_BASE_URL`, Value: Your API base URL
   - Name: `CORTEX_API_KEY`, Value: Your API key
   - Name: `CORTEX_API_KEY_ID`, Value: Your API key ID

## Step 3: Choose Your Scan Type

### Option A: Code Security Scan

Create `.github/workflows/cortex-code-scan.yml`:

```yaml
name: Cortex Code Security

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Cortex Code Scan
        uses: PaloAltoNetworks/cortex-cloud-scan@v1
        with:
          scan-type: code
          api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
          api-key: ${{ secrets.CORTEX_API_KEY }}
          api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
          directory: .
          repo-id: ${{ github.repository }}
          branch: ${{ github.ref_name }}
```

**What this scans:**
- Secrets in your code
- Infrastructure-as-Code (IaC) misconfigurations
- Dependency vulnerabilities (SCA)

### Option B: Container Image Scan

Create `.github/workflows/cortex-container-scan.yml`:

```yaml
name: Cortex Container Scan

on:
  push:
    branches: [ main ]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build image
        run: docker build -t myapp:latest .

      - name: Cortex Container Scan
        uses: PaloAltoNetworks/cortex-cloud-scan@v1
        with:
          scan-type: image
          api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
          api-key: ${{ secrets.CORTEX_API_KEY }}
          api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
          image: myapp:latest
```

**What this scans:**
- Container image vulnerabilities
- Malware detection
- SBOM generation

### Option C: API Security Scan

Create `.github/workflows/cortex-api-scan.yml`:

```yaml
name: Cortex API Security

on:
  push:
    branches: [ main ]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Cortex API Scan
        uses: PaloAltoNetworks/cortex-cloud-scan@v1
        with:
          scan-type: api
          api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
          api-key: ${{ secrets.CORTEX_API_KEY }}
          api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
          scanned-app-url: https://api.example.com
          api-spec-file: ./openapi.yaml
```

**What this scans:**
- API vulnerabilities
- Security misconfigurations
- OpenAPI/Swagger specification issues

## Step 4: Commit and Push

```bash
git add .github/workflows/cortex-*.yml
git commit -m "Add Cortex security scanning"
git push
```

## Step 5: View Results

1. Go to your repository on GitHub
2. Click the **Actions** tab
3. Select your workflow run
4. View the scan results in the workflow logs

## Next Steps

### Customize Your Scans

**Filter by severity:**
```yaml
severity: critical,high
```

**Scan specific frameworks:**
```yaml
framework: terraform,kubernetes,secrets
```

**Enable SARIF output for GitHub Security:**
```yaml
output-format: sarif
output-file-path: results.sarif
```

### Block Deployments on Findings

Set `soft-fail: false` (default) to fail the pipeline when issues are found:

```yaml
soft-fail: false
```

### Upload Results to Cortex Cloud

Use `upload-mode: upload` (default for code scans) to track findings in Cortex Cloud:

```yaml
upload-mode: upload
```

## Common Configurations

### Scan on Pull Requests Only

```yaml
on:
  pull_request:
    branches: [ main ]
```

### Scan Specific Directories

```yaml
directory: ./src
```

### Skip Certain Paths

```yaml
skip-path: node_modules,vendor
```

### Validate Detected Secrets

```yaml
validate-secrets: true
```

## Example Workflows

Check the [examples](./examples) directory for complete workflow examples:

- **[code-security-sarif.yml](./examples/code-security-sarif.yml)**: Code scan with SARIF upload to GitHub Security
- **[container-scan.yml](./examples/container-scan.yml)**: Complete container build and scan pipeline
- **[api-security-scan.yml](./examples/api-security-scan.yml)**: API security testing workflow
- **[multi-scan-pipeline.yml](./examples/multi-scan-pipeline.yml)**: Comprehensive security pipeline with all scan types

## Troubleshooting

### "Authentication failed"
- Verify your API credentials are correct
- Ensure secrets are properly configured in GitHub
- Check that your API key has the correct permissions

### "Directory not found"
- Use `directory: .` to scan the entire repository
- Ensure the path is relative to the repository root

### "Scan timeout"
- Increase timeout for large scans: `api-timeout: 900`
- For image scans: `image-timeout: 120`

## Getting Help

- **Documentation**: [README.md](./README.md)
- **Examples**: [examples/](./examples)
- **Issues**: [GitHub Issues](../../issues)
- **Support**: [SUPPORT.md](./SUPPORT.md)

## Best Practices

1. ✅ Run scans on every pull request
2. ✅ Block merges when critical issues are found
3. ✅ Upload results to Cortex Cloud for tracking
4. ✅ Use SARIF format for GitHub Security integration
5. ✅ Scan containers before pushing to registries
6. ✅ Enable secret validation to reduce false positives
7. ✅ Review and suppress known false positives
