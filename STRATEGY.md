# Python Bundle Strategy - CTO Infrastructure Plan

## The Core Problem

**Cannot build clean Python bundles on developer machines with existing Python installations.**

Your machine has:
- Python 3.13 globally installed
- User site-packages with torch, numpy, etc.
- These contaminate any bundle build attempt

## The Solution

**Build Python bundles in a clean cloud environment using GitHub Actions.**

## How It Works

### 1. Separate Bundle Repository
- **Repo**: `14-samai_python_bundle`
- **Purpose**: Build and distribute Python bundles
- **Contains**: `requirements.txt` + GitHub Actions workflow
- **Output**: Versioned releases with downloadable ZIP files

### 2. GitHub Actions Workflow
- **Trigger**: Push to `requirements.txt` or manual trigger
- **Environment**: Fresh Windows VM (no Python pre-installed)
- **Process**:
  1. Install Python 3.12 using `actions/setup-python@v5`
  2. Copy Python installation to `python-bundle/` directory
  3. Install all packages from `requirements.txt`
  4. Validate critical imports (psycopg2, lxml, anthropic, openai, etc.)
  5. Compress to ZIP
  6. Create GitHub Release with downloadable artifact

### 3. Installer Repository Integration
- **Repo**: `100-samai-desktop-installer`
- **Usage**: Download bundle from releases, extract to `bundled/python/`
- **Build**: Use clean bundle in Inno Setup installer

## What Changed in the Workflow

### OLD (Failed Approach)
```yaml
- Download WinPython .exe from GitHub
- Extract using self-extracting executable
- Find extracted Python directory
- Copy to bundle location
```
**Problem**: Extraction failed in GitHub Actions environment

### NEW (Current Approach)
```yaml
- name: Setup Python
  uses: actions/setup-python@v5
  with:
    python-version: '3.12'

- name: Create portable Python bundle
  run: |
    $pythonExe = (Get-Command python).Source
    $pythonRoot = Split-Path $pythonExe
    Copy-Item -Path "$pythonRoot\*" -Destination "python-bundle\" -Recurse -Force
```
**Fix**: Use GitHub's official Python installer, copy the installation

## Expected Outcome

1. ✅ Workflow runs successfully (no extraction errors)
2. ✅ Python 3.12 installed via `actions/setup-python`
3. ✅ All packages from `requirements.txt` installed
4. ✅ Bundle validated (all imports work)
5. ✅ Release created with downloadable `samai-python-bundle.zip`
6. ✅ You download ZIP from releases
7. ✅ Extract to installer project
8. ✅ Build installer with clean bundle

## What You Need to Do

1. **Check if workflow triggered**: https://github.com/SMEBusinessSupport/14-samai_python_bundle/actions
2. **If not running**: Manually trigger from Actions tab → "Run workflow"
3. **Monitor build**: Wait ~10-15 minutes
4. **Download release**: Go to Releases tab, download ZIP
5. **Extract to installer**: Replace `bundled/python/` with clean bundle
6. **Build installer**: Run your Inno Setup build

## Why This Approach

### Architecture Benefits
- ✅ **Clean separation**: Bundle repo independent from installer repo
- ✅ **Versioned releases**: Track bundle versions with Git tags
- ✅ **Reusability**: Other projects can use same bundles
- ✅ **Automation**: No manual VM setup, fully automated
- ✅ **Reproducibility**: Same bundle every time, guaranteed clean

### Technical Benefits
- ✅ **No contamination**: Fresh VM has no Python pre-installed
- ✅ **No manual work**: Automatic builds on requirements changes
- ✅ **Validation**: Every build tests critical imports
- ✅ **Distribution**: GitHub releases are free and fast

## Current Status

- [x] Workflow file created
- [x] Switched from WinPython to `actions/setup-python`
- [x] Committed and pushed to GitHub
- [ ] Workflow running successfully
- [ ] Release created
- [ ] Bundle downloaded and tested

## Next Steps

1. Verify workflow is running on GitHub
2. Wait for completion
3. Download release
4. Test bundle in installer
5. Document the download/integration process

---

**Bottom line**: This eliminates the contamination problem permanently by building in the cloud, not on your machine.
