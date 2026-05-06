# RetroGameDev VR — Libretro Cores

This repository hosts the [libretro](https://www.libretro.com/) emulator cores
downloaded at runtime by **RetroGameDev VR** on Meta Quest. The app fetches
`manifest.json` on launch and pulls each core from this repository the first
time a user plays a ROM that needs it.

## Cores included

| File | Systems | License | Upstream |
| --- | --- | --- | --- |
| `libmgba_libretro.so` | Game Boy, Game Boy Color, Game Boy Advance | MPL 2.0 | [mgba-emu/mgba](https://github.com/mgba-emu/mgba) |
| `libfceumm_libretro.so` | NES | GPL v2 | [libretro/libretro-fceumm](https://github.com/libretro/libretro-fceumm) |
| `libvice_x64_libretro.so` | Commodore 64 | GPL v2 or later | [libretro/vice-libretro](https://github.com/libretro/vice-libretro) |

All binaries are built for Android **arm64-v8a** (Meta Quest 3 / 3S / Pro). They
are not portable to other platforms.

## Files

- `manifest.json` — catalogue read by the app's `CoreInstaller`. Contains
  per-core version, download URL, SHA-256, and size. See
  [SOURCES.md](SOURCES.md) for the schema.
- `lib*_libretro.so` — the cores themselves.
- `LICENSE-mgba.txt`, `LICENSE-fceumm.txt`, `LICENSE-vice.txt` — full license
  text for each core.
- `SOURCES.md` — pointers to upstream source for each binary, including any
  local patches applied during the build.

## Licensing

These cores carry copyleft and weak-copyleft licenses. The corresponding source
code for each binary is currently referenced via [SOURCES.md](SOURCES.md);
pinned source archives matching the exact build commit will be added to this
repository (or its GitHub Releases) before any Store submission, in line with
GPL §3 obligations. See SOURCES.md for the current status.

## Building

These binaries are produced from the [RetroGameDev VR main repo](https://github.com/retrogamedev/retrogamedevvr)'s
`LibretroCores/Makefile`. Build flags, toolchain detection, and the VICE
patches are documented there and replayed in [SOURCES.md](SOURCES.md).

## Issues

- Bugs in the emulators themselves are best reported to each upstream project
  (see the **Upstream** column above).
- Bugs in how RetroGameDev VR uses these cores belong on the main app
  repository.
