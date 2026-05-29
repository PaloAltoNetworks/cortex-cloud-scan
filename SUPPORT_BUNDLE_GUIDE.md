# Support Bundle Guide

## What is a Support Bundle?

When you enable the `support` flag in the Cortex Cloud Scan action, cortexcli generates a compressed archive containing detailed diagnostic information about the scan execution. This bundle is invaluable for troubleshooting issues.

## When to Use Support Bundles

Enable support bundles when:

- ✅ Scans are failing with unclear error messages
- ✅ Scans are timing out unexpectedly
- ✅ You need to report an issue to Palo Alto Networks support
- ✅ Debugging performance issues
- ✅ Investigating unexpected scan results

## How to Enable

Add `support: true` to your workflow:

```yaml
- name: Cortex Scan with Support Bundle
  id: scan
  uses: PaloAltoNetworks/cortex-cloud-scan@v1
  with:
    scan-type: image
    api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
    api-key: ${{ secrets.CORTEX_API_KEY }}
    api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
    image: myapp:latest
    support: true  # ← Enable support bundle
```

## Capturing the Support Bundle

Use `actions/upload-artifact` to capture the generated bundle:

```yaml
- name: Upload support bundle
  uses: actions/upload-artifact@v4
  if: always() && steps.scan.outputs.support-bundle != ''  # Only if bundle exists
  with:
    name: cortex-support-bundle-${{ github.run_id }}
    path: ${{ steps.scan.outputs.support-bundle }}
    retention-days: 30
```

**Important**: Always check that `support-bundle` output is not empty before uploading to avoid errors.

## What's Included in a Support Bundle?

The support bundle typically contains:

- **Execution logs**: Detailed logs from cortexcli
- **System information**: OS, architecture, environment variables
- **Configuration**: Command-line flags and settings used
- **Error traces**: Stack traces and error details
- **Timing information**: Performance metrics and timing data
- **Network logs**: API calls and responses (credentials redacted)

**Note**: Sensitive information like API keys are automatically redacted.

## Complete Example Workflow

### Basic Support Bundle Capture

```yaml
name: Scan with Support Bundle

on: [push, pull_request]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build image
        run: docker build -t myapp:latest .

      - name: Scan with support bundle
        id: scan
        uses: PaloAltoNetworks/cortex-cloud-scan@v1
        continue-on-error: true
        with:
          scan-type: image
          api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
          api-key: ${{ secrets.CORTEX_API_KEY }}
          api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
          image: myapp:latest
          support: true

      - name: Upload support bundle
        uses: actions/upload-artifact@v4
        if: always() && steps.scan.outputs.support-bundle != ''
        with:
          name: support-bundle
          path: ${{ steps.scan.outputs.support-bundle }}
          retention-days: 30
```

### Conditional Support Bundle (Only on Failure)

```yaml
- name: Scan container
  id: scan
  uses: PaloAltoNetworks/cortex-cloud-scan@v1
  continue-on-error: true
  with:
    scan-type: image
    api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
    api-key: ${{ secrets.CORTEX_API_KEY }}
    api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
    image: myapp:latest
    support: ${{ failure() }}  # Only generate on failure

- name: Upload support bundle if scan failed
  uses: actions/upload-artifact@v4
  if: failure() && steps.scan.outputs.support-bundle
  with:
    name: cortex-support-bundle-failure
    path: ${{ steps.scan.outputs.support-bundle }}
```

### Multiple Scans with Unique Bundle Names

```yaml
jobs:
  scan-multiple:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        image: [app1, app2, app3]
    steps:
      - name: Scan ${{ matrix.image }}
        id: scan
        uses: PaloAltoNetworks/cortex-cloud-scan@v1
        with:
          scan-type: image
          api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
          api-key: ${{ secrets.CORTEX_API_KEY }}
          api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
          image: ${{ matrix.image }}:latest
          support: true

      - name: Upload support bundle for ${{ matrix.image }}
        uses: actions/upload-artifact@v4
        if: always() && steps.scan.outputs.support-bundle != ''
        with:
          name: support-bundle-${{ matrix.image }}
          path: ${{ steps.scan.outputs.support-bundle }}
```

## Accessing the Support Bundle

1. Go to your GitHub Actions workflow run
2. Scroll to the **Artifacts** section at the bottom
3. Click on the support bundle to download it
4. Extract the `.tar.gz` file to view contents

## Sharing with Support

If you need to share the bundle with Palo Alto Networks support:

1. Download the artifact from GitHub Actions
2. Upload to your support case portal
3. Reference the bundle in your support ticket

## File Naming Convention

Support bundles are typically named:
```
cortex-support-YYYY-MM-DD-HH-MM-SS.tar.gz
```

Example:
```
cortex-support-2026-05-29-14-30-45.tar.gz
```

## Performance Impact

Enabling support bundles has minimal performance impact:

- **Overhead**: ~1-2% additional execution time
- **File size**: Typically 1-5MB compressed
- **Storage**: Uses GitHub Actions artifact storage (part of your quota)

## Best Practices

1. ✅ **Check bundle exists before upload** - Use `if: always() && steps.scan.outputs.support-bundle != ''`
2. ✅ **Use descriptive artifact names** - Include run ID or timestamp for uniqueness
3. ✅ **Set appropriate retention** - Default to 30 days, adjust based on needs
4. ✅ **Enable only when needed** - Don't generate bundles for every successful scan
5. ✅ **Add to issue templates** - Request support bundles when users report problems

## Troubleshooting Support Bundles

**Bundle not generated:**
- Verify `support: true` is set
- Check that cortexcli completed execution (even with errors)
- Look for "Support bundle generated" message in logs

**Bundle path is empty:**
- The action looks for `cortex-support-*.tar.gz` files
- Check workflow logs for warnings about missing files
- Ensure cortexcli has write permissions in the working directory

**Artifact upload fails:**
- Check that the path from `outputs.support-bundle` is valid
- Use `if-no-files-found: warn` to see helpful error messages
- Verify GitHub Actions has artifact upload permissions

## Example: Issue Template with Support Bundle

When users report issues, request the support bundle:

```markdown
## Bug Report

**Describe the issue:**
[Your description]

**Workflow file:**
```yaml
[Paste your workflow]
```

**Support bundle:**
Please attach the support bundle:
1. Enable `support: true` in your workflow
2. Run the scan again
3. Download the artifact
4. Attach the `.tar.gz` file here
```

## Security Considerations

- ✅ API keys and secrets are automatically redacted
- ✅ Bundles may contain file paths and repository structure
- ✅ Network logs show API endpoints but not sensitive data
- ⚠️ Review bundle contents before sharing publicly
- ✅ Safe to share with Palo Alto Networks support

## Retention and Cleanup

GitHub Actions artifacts have configurable retention:

```yaml
retention-days: 30  # Keep for 30 days (default: 90)
```

Bundles are automatically deleted after the retention period. To clean up manually:

1. Go to repository Settings → Actions → General
2. Scroll to "Artifact and log retention"
3. Adjust retention or manually delete old artifacts

---

For more information, see the [main README](README.md) or [example workflows](examples/).
