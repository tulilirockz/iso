# GitHub Copilot Instructions for Bluefin ISO Repository

## Repository Purpose

The **projectbluefin/iso** repository is dedicated to building bootable installation ISOs for the Bluefin operating system. This is an ISO-only repository extracted from the main ublue-os/bluefin repository.

**Key Technologies:**
- **Titanoboa** - ISO builder
- **Anaconda** - Linux installer
- **GitHub Actions** - CI/CD automation
- **Just** - Command runner for build automation

**What this repository does:**
- Builds bootable ISOs for multiple Bluefin variants
- Configures Anaconda installer for Bluefin
- Manages flatpak application lists for pre-installation
- Automates ISO builds via GitHub Actions

**What this repository does NOT do:**
- Does NOT build base Bluefin container images (see ublue-os/bluefin)
- Does NOT manage Bluefin documentation (see ublue-os/bluefin-docs)
- Does NOT build LTS base images (see ublue-os/bluefin-lts)

## Repository Structure

```
.github/
├── workflows/
│   ├── reusable-build-iso-anaconda.yml  # Main ISO build workflow (matrix strategy)
│   ├── build-iso-lts.yml                # LTS-specific workflow caller
│   ├── validate-flatpaks.yml            # Validates flatpak lists
│   └── validate-renovate.yml            # Validates renovate config
└── renovate.json5                       # Renovate configuration

iso_files/
├── configure_iso_anaconda.sh            # Standard ISO configuration
├── configure_lts_iso_anaconda.sh        # LTS ISO configuration
└── README.md                            # ISO files documentation

flatpaks/
├── system-flatpaks.list                 # Core apps (35 flatpaks, all ISOs)
├── system-flatpaks-dx.list              # Developer tools (8 flatpaks, DX variant)
└── system-flatpaks-extra.list           # Optional apps (not currently used)

just/
├── bluefin-apps.just                    # User-facing app management
└── bluefin-system.just                  # System management recipes

Justfile                                 # Main build recipes (32KB, from ublue-os/bluefin)
AGENTS.md                                # Detailed agent instructions (387 lines)
CONTRIBUTING.md                          # Contribution guidelines
README.md                                # Repository documentation
.pre-commit-config.yaml                  # Pre-commit validation hooks
```

## ISO Variants and Flavors

### Variants (Release Channels)
- **gts** - Grand Touring Support (stable with extended support)
- **stable** - Current stable release
- **latest** - Latest features (may be unstable)
- **beta** - Pre-release testing builds
- **lts** - Long-term support (CentOS Stream-based)
- **lts-hwe** - LTS with hardware enablement kernel

### Flavors (Image Types)
- **main** - Standard Bluefin (available for all variants)
- **nvidia-open** - With NVIDIA open drivers (GTS/Stable only)
- **gdx** - Bluefin DX for developers (LTS only)

### Platforms
- **amd64** (x86_64)
- **arm64** (aarch64)

### Base Container Images
ISOs are built from pre-built container images:
- `ghcr.io/ublue-os/bluefin*`
- `ghcr.io/projectbluefin/bluefin*`

## Key Workflows

### reusable-build-iso-anaconda.yml
**Purpose:** Main ISO build workflow with matrix strategy

**Triggers:**
- Workflow dispatch (manual builds, select variant)
- Pull requests (on ISO configuration changes)
- Workflow call (from other workflows like LTS)

**Matrix Strategy:**
Dynamically generates build combinations based on variant:
- `lts`: amd64/arm64 × main/gdx
- `lts-hwe`: amd64/arm64 × main
- `gts`: amd64 × main/nvidia-open
- `stable`: amd64 × main/nvidia-open
- `all`: All platform/flavor/variant combinations

**Build Process:**
1. Determine matrix based on input variant
2. Maximize disk space (ISOs are large)
3. Setup Just command runner
4. Format container image references
5. Add Bazaar flatpak to LTS builds (automatic)
6. Build ISO using Titanoboa action
7. Rename and checksum ISO
8. Upload to CloudFlare R2 or GitHub artifacts

**Key Features:**
- Parallel builds across matrix
- Separate runners for arm64 vs amd64
- Conditional Bazaar addition for LTS
- Different builder distro: CentOS for LTS, Fedora for others
- Conditional uploads based on event type

### build-iso-lts.yml
**Purpose:** Trigger LTS ISO builds

**What it does:**
- Calls reusable workflow with LTS-specific parameters
- Builds both main and gdx flavors for LTS

### validate-flatpaks.yml
**Purpose:** Validate flatpak list files

**What it does:**
- Checks that flatpak IDs exist on Flathub
- Runs on changes to `flatpaks/*.list` files

## ISO Configuration Scripts

### configure_iso_anaconda.sh (Standard ISOs)
**For:** GTS, Stable, Latest, Beta variants

**Key Functions:**
- Removes unnecessary packages from live environment
- Configures GNOME dock with installer shortcuts
- Disables sleep/suspend during installation
- Installs Anaconda installer and WebUI (F42+)
- Creates Anaconda profile for Bluefin
- Configures automatic flatpak installation
- Sets up secure boot key enrollment
- Customizes branding and artwork

**Filesystem:** BTRFS with compression
**Bootloader:** `efi_dir = fedora`
**Secure Boot:** Enabled with key enrollment

### configure_lts_iso_anaconda.sh (LTS ISOs)
**For:** LTS, LTS-HWE variants

**Key Differences from Standard:**
- Based on CentOS Stream, not Fedora
- Uses XFS filesystem instead of BTRFS
- Different partitioning scheme
- Modified bootloader configuration: `efi_dir = centos`
- Secure boot enrollment commented out (disabled)
- CentOS-specific repository configuration

## Flatpak Management

### system-flatpaks.list (35 flatpaks)
Core applications included on ALL ISOs:
- Firefox browser
- GNOME core applications
- Essential utilities

### system-flatpaks-dx.list (8 flatpaks)
Developer tools for DX variants:
- IDEs and development utilities
- Only included on DX ISOs (LTS-gdx)

### system-flatpaks-extra.list
Optional applications, not currently used in builds

### Special Cases
- **Bazaar:** Automatically added to LTS builds by workflow (not in list files)
- Flatpak IDs are validated against Flathub via GitHub Actions

## Build System (Justfile)

The `Justfile` contains build automation recipes (32KB, copied from ublue-os/bluefin).

**Key ISO Recipes:**
- `build-iso [image] [tag] [flavor] [ghcr] [pipeline]` - Build ISO
- `build-iso-ghcr [image] [tag] [flavor]` - Build ISO from GHCR (recommended)
- `run-iso [image] [tag] [flavor]` - Test ISO in VM
- `validate [image] [tag] [flavor]` - Validate image/tag/flavor combo
- `image_name [image] [tag] [flavor]` - Get full image name
- `fedora_version [image] [tag] [flavor]` - Get Fedora version

**Key Variables:**
- `repo_organization` - GitHub organization
- `images` - Image name mappings
- `flavors` - Flavor mappings (main, nvidia-open, gdx)
- `tags` - Tag/variant mappings (gts, stable, latest, beta, lts, lts-hwe)

**Note:** Most production builds happen via GitHub Actions, not locally. Local builds require 50GB+ disk space and 30-60 minutes.

## Development Workflow

### Validation Requirements (MANDATORY)
All contributions must pass:

```bash
# 1. Pre-commit validation (syntax, formatting)
pre-commit run --all-files

# 2. Just syntax check (if Just is installed)
just check

# 3. Auto-fix formatting
just fix
```

**Pre-commit Hooks:**
- `check-json` - Validates JSON syntax
- `check-toml` - Validates TOML syntax
- `check-yaml` - Validates YAML (workflows, etc.)
- `end-of-file-fixer` - Ensures files end with newline
- `trailing-whitespace` - Removes trailing whitespace

### Common Development Tasks

**Add Flatpaks:**
1. Edit `flatpaks/system-flatpaks*.list`
2. Add one flatpak ID per line
3. Use `#` for comments
4. GitHub Actions will validate against Flathub

**Modify ISO Configuration:**
1. Edit `iso_files/configure_*_anaconda.sh`
2. Test via workflow dispatch (manual build)
3. Check the generated ISO

**Update Workflows:**
1. Edit `.github/workflows/*.yml`
2. Test in your fork first
3. Use workflow dispatch for validation
4. Follow conventional commit format

**Test Changes:**
- **Preferred:** Use workflow dispatch for manual ISO builds
- **Local builds:** Resource-intensive, use `just build-iso-ghcr` to avoid building base image
- **Validation:** Always run pre-commit before committing

### Build Strategy

**Local Builds:**
- Resource-intensive: 50GB+ disk, 30-60 minutes per ISO
- Requires privileged container access
- Use `just build-iso-ghcr` to build from GHCR images
- Better to test via GitHub Actions when possible

**CI Builds (Preferred):**
- Use workflow dispatch for manual testing
- Automatic builds on configuration changes
- Parallel matrix builds for efficiency
- Professional build environment

**Production:**
- Automatic builds on configuration changes
- Uploads to CloudFlare R2
- Checksums generated automatically

## Commit Conventions (MANDATORY)

This repository uses [Conventional Commits](https://www.conventionalcommits.org/):

**Format:** `<type>(<scope>): <description>`

**Types:**
- `feat:` - New features
- `fix:` - Bug fixes
- `docs:` - Documentation changes
- `ci:` - CI/CD changes
- `chore:` - Maintenance tasks
- `refactor:` - Code refactoring

**Examples:**
- `feat(iso): add ARM64 support`
- `fix(flatpaks): correct invalid flatpak ID`
- `docs(readme): update build instructions`
- `ci(workflow): optimize matrix strategy`

**AI Attribution:**
When using AI tools, include footer:
```
Assisted-by: [Model Name] via [Tool Name]
```

Example:
```
feat(iso): add ARM64 support

Add ARM64 platform to build matrix

Assisted-by: Claude 3.5 Sonnet via GitHub Copilot
```

## Key Differences: Standard vs LTS

| Aspect | Standard (GTS/Stable/Latest/Beta) | LTS (lts/lts-hwe) |
|--------|-----------------------------------|-------------------|
| **Base OS** | Fedora Linux | CentOS Stream |
| **Filesystem** | BTRFS with compression | XFS |
| **Builder Distro** | Fedora | CentOS |
| **Secure Boot** | Enabled with key enrollment | Disabled (commented out) |
| **Bootloader** | `efi_dir = fedora` | `efi_dir = centos` |
| **Partitioning** | Standard Fedora scheme | Modified for CentOS |
| **Repositories** | Fedora repos | CentOS repos |
| **Special Flatpaks** | None | Bazaar (auto-added by workflow) |

## Best Practices for AI Agents

### DO:
- ✅ Make minimal, surgical changes
- ✅ Focus on ISO configuration and workflows
- ✅ Use conventional commits
- ✅ Run validation before committing
- ✅ Test via workflow dispatch when possible
- ✅ Reference AGENTS.md for detailed instructions
- ✅ Use existing patterns and conventions
- ✅ Include AI attribution in commits

### DON'T:
- ❌ Modify base image building (wrong repo)
- ❌ Remove or edit working code unnecessarily
- ❌ Fix unrelated bugs or broken tests
- ❌ Build ISOs locally unless necessary
- ❌ Skip validation steps
- ❌ Use non-conventional commit messages
- ❌ Add new tools without justification

### Common Pitfalls:
- **Large builds:** ISOs are resource-intensive, prefer GitHub Actions
- **Pre-commit failures:** Run `pre-commit run --all-files` before commit
- **Just syntax errors:** Run `just check` and `just fix`
- **Workflow testing:** Test in fork first with workflow dispatch
- **Flatpak validation:** IDs are checked against Flathub automatically

## Related Resources

- **Main Bluefin repo:** https://github.com/ublue-os/bluefin
- **Bluefin LTS repo:** https://github.com/ublue-os/bluefin-lts
- **Bluefin docs:** https://github.com/ublue-os/bluefin-docs
- **Titanoboa:** https://github.com/ublue-os/titanoboa
- **Detailed agent instructions:** See `AGENTS.md` in this repository

## Quick Reference

### File Locations
- Workflows: `.github/workflows/`
- ISO configs: `iso_files/configure_*_anaconda.sh`
- Flatpaks: `flatpaks/*.list`
- Build recipes: `Justfile`, `just/*.just`
- Documentation: `README.md`, `AGENTS.md`, `CONTRIBUTING.md`

### Common Commands
```bash
# Validation
pre-commit run --all-files
just check
just fix

# Local ISO build (from GHCR)
just build-iso-ghcr bluefin stable main

# Test ISO in VM
just run-iso bluefin stable main

# Clean build artifacts
just clean
```

### Matrix Variants
- **lts:** amd64/arm64 × main/gdx
- **lts-hwe:** amd64/arm64 × main
- **gts:** amd64 × main/nvidia-open
- **stable:** amd64 × main/nvidia-open
- **all:** All combinations

## Summary

This repository is focused exclusively on ISO generation for Bluefin. It uses pre-built container images and configures them for bootable installation media using Anaconda and Titanoboa. Development should focus on ISO configuration, workflows, and flatpak management. Always validate changes, use conventional commits, and prefer GitHub Actions for testing over local builds.
