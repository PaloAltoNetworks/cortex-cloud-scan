# Cortexcli Installation Method

## How cortexcli is Installed

This GitHub Action downloads cortexcli **directly from the Cortex API** using your credentials, not from GitHub releases or Homebrew.

### Installation Flow

1. **Detect Platform**: Determine OS (linux/darwin) and architecture (amd64/arm64)
2. **Request Download Link**: Call the Cortex API endpoint to get a signed download URL:
   ```
   GET https://<api-host>/public_api/v1/unified-cli/releases/download-link?os=<os>&architecture=<arch>
   Headers:
     - x-xdr-auth-id: <API_KEY_ID>
     - Authorization: <API_KEY>
   ```
3. **Download Binary**: Use the signed URL to download the cortexcli tarball
4. **Extract & Install**: Extract the binary and move to `/usr/local/bin/`

### Why This Method?

- ✅ **Always Latest**: Gets the version compatible with your Cortex tenant
- ✅ **Secure**: Uses your existing API credentials
- ✅ **Authenticated**: Requires valid Cortex API access
- ✅ **Platform Specific**: Downloads the correct binary for the runner OS/architecture

### API Response Format

```json
{
  "signed_url": "https://...",
  "file_name": "cortexcli-v1.2.3-linux-amd64.tar.gz"
}
```

### Supported Platforms

The action automatically detects and downloads the correct binary for:

- **Linux**: amd64, arm64
- **macOS**: amd64, arm64

These match GitHub-hosted runner architectures:
- `ubuntu-latest`: linux/amd64
- `ubuntu-24.04-arm`: linux/arm64 (preview)
- `macos-latest`: darwin/amd64 or darwin/arm64

### Requirements

Your API credentials must have permission to access the unified-cli download endpoint. This is typically included in standard CLI tool access permissions.

### Troubleshooting

**"Failed to get download URL"**
- Check API credentials are correct
- Verify api-base-url format: `https://api-<tenant>.xdr.<region>.paloaltonetworks.com`
- Ensure API key has CLI download permissions

**"Unsupported architecture"**
- The runner architecture is not supported
- Use standard GitHub-hosted runners (ubuntu-latest, macos-latest)

**"tar: not in gzip format"**
- This was the original error - fixed by using the Cortex API method
- The signed URL endpoint returns valid tarballs

### Example Installation Command (for reference)

This is what happens inside the action:

```bash
# Get signed download URL
RESPONSE=$(curl -s "https://api-tenant.xdr.us.paloaltonetworks.com/public_api/v1/unified-cli/releases/download-link?os=linux&architecture=amd64" \
  -H "x-xdr-auth-id: $CORTEX_API_KEY_ID" \
  -H "Authorization: $CORTEX_API_KEY")

# Extract URL and filename
SIGNED_URL=$(echo "$RESPONSE" | jq -r '.signed_url')
FILE_NAME=$(echo "$RESPONSE" | jq -r '.file_name')

# Download and install
curl -L -o "$FILE_NAME" "$SIGNED_URL"
tar -xzf "$FILE_NAME"
chmod +x cortexcli
sudo mv cortexcli /usr/local/bin/
```

### No Version Pinning

Unlike typical GitHub Actions that download from GitHub releases, this action always downloads the latest version available for your Cortex tenant. This is **intentional** because:

1. Cortex CLI versions are tied to your tenant version
2. The Cortex API always provides the compatible version
3. No manual version management needed

If you need a specific version, you would need to download it outside this action and provide the path (feature not currently implemented).

### Dependencies

The installation script uses:
- `curl` - Download files (available on all GitHub runners)
- `tar` - Extract archives (available on all GitHub runners)
- `jq` - JSON parsing (installed on GitHub runners, fallback to grep/sed if missing)
- `sudo` - Move binary to system path (available on all GitHub runners)

All dependencies are standard on GitHub-hosted runners.
