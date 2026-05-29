# Action Reference Guide

## How to Reference This Action

### Local Development (Before Publishing)

When testing in the same repository:

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: ./  # References action.yml in repository root
    with:
      scan-type: code
      # ... inputs
```

### After Publishing to GitHub

Once pushed to a public GitHub repository:

```yaml
# Recommended: Use major version tag (gets updates)
- uses: PaloAltoNetworks/cortex-cloud-scan@v1

# Specific version (locked, no automatic updates)
- uses: PaloAltoNetworks/cortex-cloud-scan@v1.0.0

# Specific commit SHA (most secure, immutable)
- uses: PaloAltoNetworks/cortex-cloud-scan@abc123def

# Branch reference (NOT recommended for production)
- uses: PaloAltoNetworks/cortex-cloud-scan@main
```

## Version Tag Strategy

### Major Version Tags (Recommended)

```bash
# Create release
git tag v1.0.0
git push origin v1.0.0

# Update major version pointer
git tag -fa v1 -m "Update v1 to v1.0.0"
git push origin v1 --force
```

**Users reference:** `@v1`
**Benefit:** Automatically get minor updates and patches

### Full Version Tags

```bash
git tag v1.2.3
git push origin v1.2.3
```

**Users reference:** `@v1.2.3`
**Benefit:** Locked to exact version, no surprises

## Action Type: Composite

This action is built as a **composite action** using shell scripts.

### Why Composite?

| Aspect | This Action (Composite) | JavaScript Alternative |
|--------|------------------------|------------------------|
| **Purpose** | CLI wrapper | Complex logic |
| **Maintenance** | ✅ Simple YAML | Requires build process |
| **Dependencies** | None | npm packages, compilation |
| **Startup Time** | ~3-5 seconds | ~1-2 seconds |
| **Complexity** | ✅ Low | Medium/High |
| **Transparency** | ✅ Easy to audit | Requires reviewing compiled code |
| **Updates** | ✅ Edit YAML directly | Rebuild required |

### Composite Action Structure

```yaml
# action.yml
runs:
  using: 'composite'  # ← Composite action
  steps:
    - name: Install CLI
      shell: bash
      run: |
        # Shell commands to install cortexcli

    - name: Run scan
      shell: bash
      env:
        # Environment variables
      run: |
        # Shell commands to execute scan
```

**No compilation or build step required!**

## When Would You Use JavaScript?

Consider rewriting as a JavaScript action if:

1. **Complex Logic**: Need if/else, loops, data transformations
2. **API Calls**: Direct HTTP requests, authentication flows
3. **npm Packages**: Need specific Node.js libraries
4. **Performance Critical**: Action runs thousands of times
5. **Advanced Error Handling**: Need granular error catching
6. **State Management**: Need to persist data between steps

**For wrapping cortexcli: Composite is perfect! ✅**

## Publishing to Marketplace

### Requirements

1. ✅ Public repository
2. ✅ Valid `action.yml` with metadata
3. ✅ LICENSE file
4. ✅ README.md documentation
5. ✅ Branding metadata (icon + color)

### Publishing Steps

```bash
# 1. Create release tag
git tag -a v1.0.0 -m "Initial release"
git push origin v1.0.0

# 2. Create major version tag
git tag -fa v1 -m "v1.0.0"
git push origin v1 --force

# 3. Go to GitHub → Releases → Create release
# 4. Check "Publish to GitHub Marketplace"
# 5. Choose category (Security)
# 6. Publish
```

See [PUBLISHING.md](PUBLISHING.md) for complete guide.

## Referencing Different Versions

### Scenario 1: Always Want Latest Features

```yaml
uses: PaloAltoNetworks/cortex-cloud-scan@v1
```
- Gets v1.0.0, v1.1.0, v1.2.0 automatically
- Won't get v2.0.0 (breaking changes)

### Scenario 2: Stability is Critical

```yaml
uses: PaloAltoNetworks/cortex-cloud-scan@v1.0.0
```
- Locked to exactly v1.0.0
- Never changes

### Scenario 3: Maximum Security (Immutable)

```yaml
uses: PaloAltoNetworks/cortex-cloud-scan@abc123def456
```
- References exact commit SHA
- Cannot be changed (even if tag is moved)

### Scenario 4: Development/Testing

```yaml
uses: PaloAltoNetworks/cortex-cloud-scan@main
```
- Always latest commit on main branch
- ⚠️ NOT recommended for production

## Usage Examples by Reference Type

### In Organization (Before Public Release)

```yaml
# Reference from another private repo in same org
- uses: PaloAltoNetworks/cortex-cloud-scan@feat-initial
  with:
    scan-type: code
    # ...
```

### After Marketplace Publication

```yaml
# Public users reference published action
- uses: PaloAltoNetworks/cortex-cloud-scan@v1
  with:
    scan-type: code
    api-base-url: ${{ secrets.CORTEX_API_BASE_URL }}
    api-key: ${{ secrets.CORTEX_API_KEY }}
    api-key-id: ${{ secrets.CORTEX_API_KEY_ID }}
    directory: .
```

## Migration Path

### Phase 1: Development
```yaml
# Local testing
uses: ./
```

### Phase 2: Internal Use
```yaml
# Push to GitHub, use branch
uses: PaloAltoNetworks/cortex-cloud-scan@feat-initial
```

### Phase 3: Initial Release
```yaml
# Create v1.0.0 release
uses: PaloAltoNetworks/cortex-cloud-scan@v1.0.0
```

### Phase 4: Production
```yaml
# Use major version tag
uses: PaloAltoNetworks/cortex-cloud-scan@v1
```

## Comparison: Composite vs JavaScript Actions

### Example Composite Action (This Project)

```yaml
# action.yml
name: My Action
runs:
  using: composite
  steps:
    - run: echo "Hello"
      shell: bash
```

**File structure:**
```
.
├── action.yml          ← Single file!
├── README.md
└── LICENSE
```

**Deployment:** Push and release. Done! ✅

### Example JavaScript Action

```javascript
// index.js
const core = require('@actions/core');
const exec = require('@actions/exec');

async function run() {
  try {
    const input = core.getInput('my-input');
    await exec.exec('cortexcli', ['scan', input]);
  } catch (error) {
    core.setFailed(error.message);
  }
}

run();
```

**File structure:**
```
.
├── action.yml          ← Points to dist/index.js
├── src/
│   └── index.ts       ← TypeScript source
├── dist/
│   └── index.js       ← Compiled JavaScript (must commit!)
├── node_modules/       ← Dependencies
├── package.json
├── tsconfig.json
└── README.md
```

**Deployment:** 
1. Write TypeScript
2. Compile to JavaScript
3. Bundle dependencies with ncc
4. Commit dist/ folder
5. Push and release

**More complex, but offers more flexibility for complex logic.**

## Best Practices

### For Action Developers

1. ✅ Use major version tags (`@v1`)
2. ✅ Never force-push to version tags (except major versions)
3. ✅ Follow semantic versioning
4. ✅ Keep action.yml simple and readable
5. ✅ Provide comprehensive examples
6. ✅ Test before releasing

### For Action Users

1. ✅ Use major version tags for flexibility (`@v1`)
2. ✅ Use exact versions for critical pipelines (`@v1.0.0`)
3. ✅ Use commit SHAs for maximum security
4. ✅ Never use `@main` in production
5. ✅ Review action code before using
6. ✅ Pin versions in production workflows

## Common Questions

**Q: Should I use composite or JavaScript?**
A: For CLI wrappers like this, composite is simpler and better. Use JavaScript for complex logic.

**Q: Do I need to compile anything?**
A: No! Composite actions need no compilation. Just push and release.

**Q: How do users reference my action?**
A: `owner/repo@version`, e.g., `PaloAltoNetworks/cortex-cloud-scan@v1`

**Q: Can I update the action after release?**
A: Yes, create new releases. Users on `@v1` get updates automatically.

**Q: How do I test before publishing?**
A: Use local reference `./` or push to a branch and reference `@branch-name`

## Resources

- 📖 [Publishing Guide](PUBLISHING.md)
- 🚀 [Quick Start](QUICKSTART.md)
- 📚 [Examples](examples/)
- 🔧 [GitHub Actions Docs](https://docs.github.com/en/actions)
