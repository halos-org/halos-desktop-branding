# halos-desktop-branding

HaLOS branding and default wallpaper for HaLOS desktop variants. Ships a single Debian package, `halos-desktop-branding`, that overrides the upstream Raspberry Pi OS default desktop wallpaper with a HaLOS-branded one.

Sibling package: [`hatlabs/halos-halpi-desktop-branding`](https://github.com/hatlabs/halos-halpi-desktop-branding) — same mechanism, HALPI2 artwork.

## What it ships

- `/usr/share/halos/wallpapers/halos.jpg` — the wallpaper image.
- `/etc/xdg/pcmanfm/default/desktop-items-0.conf` and `-1.conf` — pcmanfm system-wide desktop configuration pointing at the wallpaper.

## How the override works

The two `.conf` paths are owned by the upstream `rpd-common` package. This package uses `dpkg-divert` in its `preinst` to redirect `rpd-common`'s versions to `.upstream` and ship its own at the canonical paths. Future `rpd-common` upgrades land at the diverted paths, so the HaLOS default is preserved.

Per-user wallpaper choices in `~/.config/pcmanfm/default/desktop-items-{0,1}.conf` take precedence — user agency is preserved.

## Exclusivity with HALPI2 branding

`halos-desktop-branding` declares `Provides: halos-desktop-wallpaper`, `Conflicts: halos-desktop-wallpaper`, `Replaces: halos-desktop-wallpaper`. The HALPI2 variant does the same. Only one of the two real packages can be installed at a time; the `halos-desktop` metapackage depends on the virtual `halos-desktop-wallpaper` name.

## Development

```bash
./run build-debtools  # first time
./run build-deb
./run lint-deb
```

See `AGENTS.md` for the full development guide.

## Source artwork

The `sources/` directory contains the Affinity Photo source file and a lossless PNG export, tracked via Git LFS and excluded from the `.deb`. Install Git LFS (`brew install git-lfs && git lfs install`) before cloning if you intend to edit the artwork.

## License

Code: MIT. Wallpaper artwork: CC-BY-4.0 © Hat Labs Ltd.
