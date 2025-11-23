# SAM AI Python Bundle

**Clean, isolated Python environment for SAM AI Odoo 18 installer**

Built in **clean GitHub Actions environment** to guarantee zero contamination from developer machines.

---

## What This Is

This repository builds a complete Python 3.12 bundle with all dependencies for:
- **Odoo 18** (full server dependencies)
- **SAM AI** (anthropic, openai, chromadb, sentence-transformers, GitPython)

**The bundle is built automatically in a fresh Windows VM** - no local Python installations, no user packages, no contamination.

---

## Quick Start

### For Installer Developers

1. **Download latest bundle:**
   - Go to [Releases](../../releases)
   - Download `samai-python-bundle.zip` from latest release

2. **Extract to installer project:**
   ```powershell
   # Extract
   Expand-Archive -Path "samai-python-bundle.zip" -DestinationPath "C:\Temp\python"

   # Copy to installer bundled directory
   Copy-Item -Path "C:\Temp\python\*" `
             -Destination "YOUR-INSTALLER-REPO\bundled\python\" `
             -Recurse -Force
   ```

3. **Build installer** - Python bundle is now ready

---

## How It Works

### Automatic Build Process

1. **Trigger:** Push to `requirements.txt` or manual trigger
2. **Environment:** Fresh Windows VM (GitHub Actions)
3. **Process:**
   - Download WinPython 3.12.10.1
   - Extract Python distribution
   - Install all packages from `requirements.txt`
   - Validate all critical imports
   - Compress bundle
   - Create GitHub Release with downloadable ZIP

4. **Output:** Versioned release with clean Python bundle

### Why This Approach

**Problem:** Building Python bundle on developer machines causes contamination
- Developer machine has Python 3.13 installed
- User packages interfere with bundled packages
- Cannot guarantee clean environment

**Solution:** Build in cloud on fresh VM
- GitHub provides clean Windows VM
- No Python pre-installed
- Complete isolation guaranteed
- Reproducible builds

---

## Usage

### Option A: Download from Releases (Recommended)

```powershell
# Download latest bundle
$repo = "YOUR-ORG/14-samai_python_bundle"
$latestRelease = Invoke-RestMethod "https://api.github.com/repos/$repo/releases/latest"
$bundleUrl = $latestRelease.assets[0].browser_download_url

Invoke-WebRequest $bundleUrl -OutFile "samai-python-bundle.zip"
Expand-Archive "samai-python-bundle.zip" -DestinationPath "bundled\python"
```

### Option B: Trigger New Build

1. Go to [Actions](../../actions)
2. Click "Build and Release Python Bundle"
3. Click "Run workflow"
4. Wait ~15 minutes
5. Download from new release

---

## Updating Requirements

### Add New Python Package

1. Edit `requirements.txt`:
   ```bash
   cd "D:\SAMAI-18-SaaS\github-repos\14-samai_python_bundle"

   # Add package to requirements.txt
   echo "new-package==1.0.0" >> requirements.txt
   ```

2. Commit and push:
   ```bash
   git add requirements.txt
   git commit -m "Add new-package to requirements"
   git push
   ```

3. **GitHub automatically builds new bundle** and creates release

4. Download new release and use in installer

---

## Bundle Contents

### Base Distribution
- **WinPython 3.12.10.1** (portable Python for Windows)
- Pre-configured pip, setuptools, wheel
- Complete Python standard library

### Odoo 18 Dependencies
All packages from `requirements.txt` including:
- psycopg2-binary (PostgreSQL)
- lxml (XML processing)
- Pillow (Image processing)
- Werkzeug, Jinja2, Babel (Web framework)
- reportlab (PDF generation)
- And 90+ more Odoo dependencies

### SAM AI Packages
- **anthropic** - Claude API client
- **openai** - OpenAI API client
- **chromadb** - Vector database
- **sentence-transformers** - Embeddings
- **GitPython** - Git integration

---

## Versioning

Releases are tagged with format: `v3.12.10.1-odoo18-YYYYMMDD-HHMMSS`

Example: `v3.12.10.1-odoo18-20251123-140530`

- `3.12.10.1` - WinPython version
- `odoo18` - Target Odoo version
- `YYYYMMDD-HHMMSS` - Build timestamp

---

## File Structure

```
14-samai_python_bundle/
├── .github/
│   └── workflows/
│       └── build-and-release.yml    # GitHub Actions workflow
├── requirements.txt                  # Python packages to install
└── README.md                         # This file
```

**No actual Python files in repo!**

The bundle is built in GitHub Actions and distributed as releases.

---

## Validation

Every build validates critical imports:

```
✓ psycopg2      - PostgreSQL driver
✓ lxml          - XML processing
✓ Pillow        - Image processing
✓ werkzeug      - WSGI utility
✓ jinja2        - Template engine
✓ babel         - Internationalization
✓ anthropic     - Claude API
✓ openai        - OpenAI API
✓ chromadb      - Vector database
✓ sentence_transformers - Embeddings
✓ git (GitPython) - Git integration
```

If ANY package fails import, the build fails and no release is created.

---

## Troubleshooting

### Build fails in GitHub Actions

1. Check [Actions](../../actions) tab
2. Click failed workflow
3. Expand failed step to see error
4. Common issues:
   - Package not available for Python 3.12 → Update `requirements.txt`
   - Package build requires C compiler → May need to pre-compile or use binary wheel

### Bundle size too large

Current bundle: ~500MB compressed, ~1.5GB uncompressed

This is normal for a complete Python environment with all dependencies.

To reduce size:
- Use Git LFS in installer repo
- Host bundle on external storage
- Download on-demand during installation

### Package missing after installation

Bundle was validated before release. If package missing:
1. Check if it's in `requirements.txt`
2. Check release notes for package list
3. Trigger new build if requirements changed

---

## Cost

**GitHub Actions Free Tier:**
- Public repos: 2,000 minutes/month
- Windows VMs: 2x multiplier (1,000 actual minutes)
- Each build: ~15 minutes
- **~65 free builds per month**

More than enough for typical usage.

---

## Integration with Installer

### In installer build script:

```powershell
# Download latest Python bundle
$bundleRepo = "YOUR-ORG/14-samai_python_bundle"
$latestRelease = Invoke-RestMethod "https://api.github.com/repos/$bundleRepo/releases/latest"
$bundleUrl = $latestRelease.assets[0].browser_download_url

Write-Host "Downloading Python bundle from: $($latestRelease.name)"
Invoke-WebRequest $bundleUrl -OutFile "python-bundle.zip"

# Extract
Expand-Archive "python-bundle.zip" -DestinationPath "bundled\python" -Force

# Validate
.\scripts\diagnose_bundle_completeness.ps1

# Build installer
.\build_complete_installer.ps1
```

---

## Why Separate Repo

**Benefits:**
1. **Clean separation** - Bundle management independent of installer
2. **Version control** - Tag and track bundle versions
3. **Reusability** - Other projects can use same bundle
4. **Git size** - Installer repo stays small
5. **CI/CD simplicity** - Each repo has focused workflow

---

## Architecture

```
┌─────────────────────────────────────┐
│  14-samai_python_bundle (This Repo) │
│  - requirements.txt                 │
│  - GitHub Actions workflow          │
│  - Builds in clean VM               │
│  - Creates versioned releases       │
└──────────────┬──────────────────────┘
               │
               │ Downloads bundle ZIP from release
               ▼
┌─────────────────────────────────────┐
│  100-samai-desktop-installer        │
│  - Build script downloads bundle    │
│  - Extracts to bundled/python/      │
│  - Validates completeness           │
│  - Compiles installer               │
└─────────────────────────────────────┘
```

---

## License

Same as SAM AI project

---

## Contributing

To add/update Python packages:

1. Fork this repo
2. Edit `requirements.txt`
3. Create pull request
4. After merge, GitHub automatically builds new bundle

---

**This is the correct architectural pattern for managing Python bundles.**

**No more contamination. No more manual builds. Automated, clean, versioned releases.**
