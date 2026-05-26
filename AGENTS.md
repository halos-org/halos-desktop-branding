# halos-desktop-branding - Agentic Coding Guide

**LAST MODIFIED**: 2026-05-26

**Document Purpose**: Guide for AI assistants working on halos-desktop-branding.

## For Agentic Coding: Use the HaLOS Workspace

When using Claude Code or other AI assistants, work from the halos workspace repository for full context across all HaLOS repositories.

## About This Project

HaLOS-branded default desktop wallpaper for HaLOS desktop variants. Ships:

- **`/usr/share/halos/wallpapers/halos.jpg`** — the wallpaper image.
- **`/etc/xdg/pcmanfm/default/desktop-items-{0,1}.conf`** — pcmanfm system-wide desktop configuration, pointing at the wallpaper.

The two conffiles are installed at paths owned by the upstream `rpd-common` package. `debian/preinst` adds a `dpkg-divert` for each, redirecting `rpd-common`'s versions to `.upstream` so future upgrades of `rpd-common` do not clobber our default. `debian/postrm` reverses the diversions on `remove`/`purge`.

The package declares `Provides: halos-desktop-wallpaper`, `Conflicts: halos-desktop-wallpaper`, `Replaces: halos-desktop-wallpaper`. This virtual package is depended on by `halos-desktop` (in `halos-metapackages`) and is also Provided by `hatlabs/halos-halpi-desktop-branding`; only one real provider can be installed at a time. Pi-gen pre-installs the HALPI2 variant on HALPI2 desktop builds; on other desktop builds, this package is the only candidate and apt picks it.

Both real providers ship their respective wallpaper to the same path (`/usr/share/halos/wallpapers/halos.jpg`). The path collision is enforced at the dpkg-solver level via the virtual-package Conflicts above, not at the filesystem level — `apt` refuses to install both packages simultaneously, so the two `halos.jpg` files never need to coexist on disk.

Per-user wallpaper choices in `~/.config/pcmanfm/default/desktop-items-{0,1}.conf` take precedence over the system defaults this package ships, so user agency is preserved.

## Repository Layout

```
.
├── debian/                        Debian packaging
├── docker/                        debtools container for building
├── etc/xdg/pcmanfm/default/       Conffiles, mirror of install layout
├── usr/share/halos/wallpapers/    Asset(s) shipped in the .deb
├── sources/                       Affinity Photo + PNG sources (LFS, not shipped)
├── .github/                       CI workflows, local actions, lefthook scripts
├── run                            Build/lint/deploy commands
├── VERSION                        Single source of truth for package version
├── .bumpversion.cfg               bumpversion config (commits VERSION + dch)
└── lefthook.yml                   Pre-commit hooks
```

Source files (`*.afphoto`, `*.png`) under `sources/` are tracked via Git LFS and excluded from the `.deb` by `debian/halos-desktop-branding.install` (only the JPG path under `usr/share/halos/wallpapers/` is listed).

## Git Workflow Policy

**MANDATORY**: PRs must ALWAYS have all checks passing before merging. No exceptions.

**Branch Workflow:** Never push to main directly — always use feature branches and PRs.

**Changelog Policy**: Never edit `debian/changelog` directly. Always use `./run bumpversion` which uses `dch` for proper RFC 2822 date formatting.

**VERSION bumps**: Per release cycle, not per PR. See workspace `AGENTS.md` for the policy.

## Quick Start

```bash
# Build package
./run build-debtools  # First time only — builds the debtools container
./run build-deb

# Check quality
./run lint-deb

# Deploy to test device
./run deploy pi@halosdev.local

# Clean build artifacts
./run clean
```

## Verification on a Live Device

```bash
# Install the package
sudo apt install halos-desktop-branding

# Confirm diversions in place
dpkg-divert --list halos-desktop-branding

# Confirm our conffile is at the canonical path
dpkg -S /etc/xdg/pcmanfm/default/desktop-items-0.conf
# -> halos-desktop-branding

# Upstream's file should be at the diverted path
ls /etc/xdg/pcmanfm/default/desktop-items-0.conf.upstream
```

After a labwc session restart, the HaLOS wallpaper should be visible on the desktop.
