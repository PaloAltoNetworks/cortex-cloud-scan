# Publishing Guide

This guide explains how to publish the Cortex Cloud Scan GitHub Action to the GitHub Marketplace.

## Prerequisites

- Repository must be public
- You must have admin access to the repository
- Action must have proper metadata in `action.yml`

## Why Composite Action is the Right Choice

This action uses **composite** (shell-based) approach rather than JavaScript/TypeScript because:

✅ **Simpler Maintenance** - No build process, dependencies, or compilation
✅ **Transparency** - Shell commands are easy to review and audit
✅ **Perfect for CLI Wrappers** - Just orchestrating an existing binary
✅ **No Runtime Dependencies** - Doesn't require Node.js modules
✅ **Easy to Update** - Modify shell commands directly
✅ **Community Friendly** - Contributors understand shell scripts

### When Would You Use JavaScript?

JavaScript actions are better when you need:
- Complex logic with conditionals and loops
- Direct API integrations
- npm package functionality
- Faster startup time (milliseconds vs seconds)
- Advanced error handling and state management

**For this use case (wrapping cortexcli), composite is ideal.**

## Publishing Steps

### 1. Prepare the Repository

Ensure these files are present and up-to-date:

```bash
# Required files
✓ action.yml              # Action definition
✓ README.md               # Main documentation
✓ LICENSE                 # License file
✓ CHANGELOG.md            # Version history

# Recommended files
✓ QUICKSTART.md           # Getting started guide
✓ CONTRIBUTING.md         # Contribution guidelines
✓ SUPPORT.md              # Support information
✓ examples/               # Example workflows
✓ .github/workflows/      # CI/CD workflows
```

### 2. Verify action.yml Metadata

The `action.yml` must include:

```yaml
name: 'Cortex Cloud Scan'                    # ✓ Required
description: 'Run Cortex Cloud security...'  # ✓ Required
author: 'Palo Alto Networks'                 # ✓ Required

branding:                                     # ✓ Required for Marketplace
  icon: 'shield'                              # Choose from Feather icons
  color: 'orange'                             # Choose: white, yellow, blue, etc.
```

### 3. Test Locally First

Before publishing, test the action locally:

```yaml
# In your test workflow
steps:
  - uses: actions/checkout@v4
  - uses: ./  # Reference local action
    with:
      scan-type: code
      # ... other inputs
```

### 4. Create a Release

#### Option A: Using GitHub CLI

```bash
# Tag the release
git tag -a v1.0.0 -m "Initial release"
git push origin v1.0.0

# Create release
gh release create v1.0.0 \
  --title "Cortex Cloud Scan v1.0.0" \
  --notes-file CHANGELOG.md

# Create/update major version tag
git tag -fa v1 -m "Update v1 to v1.0.0"
git push origin v1 --force
```

#### Option B: Using GitHub Web UI

1. Go to your repository on GitHub
2. Click **Releases** → **Draft a new release**
3. Click **Choose a tag** → Type `v1.0.0` → **Create new tag**
4. Set release title: `Cortex Cloud Scan v1.0.0`
5. Add release notes from CHANGELOG.md
6. Click **Publish release**

### 5. Publish to Marketplace

#### First Time Publishing:

1. Navigate to your repository on GitHub
2. Go to **Releases** and find your release
3. Click **Draft a new release** or edit existing release
4. **Check the box** "Publish this Action to the GitHub Marketplace"
5. GitHub will validate your action.yml
6. Choose a **category**:
   - Recommended: **Security**
   - Alternative: **Code Quality** or **Continuous Integration**
7. Review the marketplace listing preview
8. Click **Publish release**

#### Subsequent Updates:

1. Create a new release with a new version tag
2. The marketplace listing will automatically update
3. Users on `@v1` will get updates automatically
4. Users on `@v1.0.0` will stay on that exact version

### 6. Versioning Strategy

Use semantic versioning (semver):

```
v1.0.0 - Major.Minor.Patch
```

**Major Version Tags** (Recommended for users):
```bash
git tag -fa v1 -m "Update v1"
git push origin v1 --force
```

Users reference as:
- `@v1` - Always latest v1.x.x (recommended)
- `@v1.0.0` - Exact version (stable)
- `@main` - Latest commit (not recommended for production)

### 7. Update Documentation

After publishing, update README.md usage examples to reference the published action:

```yaml
# Change from:
uses: ./

# To:
uses: PaloAltoNetworks/cortex-cloud-scan@v1
```

## Automated Release Process

This repository includes a `.github/workflows/release.yml` that automatically:

1. ✅ Validates the action
2. ✅ Creates GitHub release
3. ✅ Updates major version tag
4. ✅ Generates release notes

To use it:

```bash
# Create and push a version tag
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0

# The workflow will handle the rest
```

## Marketplace Listing Details

When publishing, provide:

**Short Description:**
```
Run Cortex Cloud security scans for Code Security, Container Workload Protection, and API Security
```

**Category:**
- Primary: Security
- Secondary: Continuous Integration

**Tags:**
```
security, scanning, containers, code-analysis, vulnerability-scanning
```

## Post-Publication Checklist

- [ ] Verify action appears in GitHub Marketplace
- [ ] Test the action using published version `@v1`
- [ ] Update repository README to use published reference
- [ ] Add marketplace badge to README
- [ ] Announce release (optional)

### Add Marketplace Badge

Add this to your README.md:

```markdown
[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Cortex%20Cloud%20Scan-blue.svg?colorA=24292e&colorB=0366d6&style=flat&longCache=true&logo=github)](https://github.com/marketplace/actions/cortex-cloud-scan)
```

## Testing the Published Action

Create a test repository and workflow:

```yaml
name: Test Published Action

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Test Cortex Scan
        uses: PaloAltoNetworks/cortex-cloud-scan@v1
        with:
          scan-type: code
          api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
          api-key: ${{ secrets.CORTEX_API_KEY }}
          api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
          directory: .
          upload-mode: no-upload
```

## Updating the Action

When releasing updates:

1. Update CHANGELOG.md with changes
2. Create new version tag (e.g., v1.1.0)
3. Create GitHub release
4. Update major version tag:
   ```bash
   git tag -fa v1 -m "Update v1 to v1.1.0"
   git push origin v1 --force
   ```

Users on `@v1` will automatically get the update.

## Deprecation Process

If you need to deprecate a version:

1. Update README with deprecation notice
2. Create a deprecation release:
   ```bash
   git tag -a v1.2.0-deprecated -m "Deprecated: Use v2 instead"
   ```
3. Update marketplace listing description
4. Provide migration guide in CHANGELOG.md

## Support After Publishing

- Respond to issues in GitHub Issues
- Monitor GitHub Discussions (if enabled)
- Track usage with GitHub Insights
- Update based on user feedback

## Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Publishing Actions to Marketplace](https://docs.github.com/en/actions/creating-actions/publishing-actions-in-github-marketplace)
- [Composite Actions](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)
- [Semantic Versioning](https://semver.org/)
