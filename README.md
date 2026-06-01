# RetroGameDev VR — Libretro Cores

This repository hosts the [libretro](https://www.libretro.com/) emulator cores
downloaded at runtime by **RetroGameDev VR** on Meta Quest. The app fetches
`manifest.json` on launch and pulls each core from this repository the first
time a user plays a ROM that needs it.

All binaries and source archives in this repository are version **1.0**.

## Cores included

| File | Systems | License | Upstream |
| --- | --- | --- | --- |
| `libmgba_libretro.so` | Game Boy, Game Boy Color, Game Boy Advance | MPL 2.0 | [mgba-emu/mgba](https://github.com/mgba-emu/mgba) |
| `libfceumm_libretro.so` | NES | GPL v2 | [libretro/libretro-fceumm](https://github.com/libretro/libretro-fceumm) |
| `libvice_x64_libretro.so` | Commodore 64 | GPL v2 or later | [libretro/vice-libretro](https://github.com/libretro/vice-libretro) |

All binaries are built for Android **arm64-v8a** (Meta Quest 3 / 3S / Pro). They
are not portable to other platforms.

## Repository layout

- `manifest.json` — catalogue read by the app's `CoreInstaller`. Contains
  per-core version, download URL, SHA-256, size, source URL, and license
  URL. See [SOURCES.md](SOURCES.md) for the schema.
- `lib*_libretro.so` — the three core binaries.
- `LICENSE-mgba.txt`, `LICENSE-fceumm.txt`, `LICENSE-vice.txt` — full
  license text for each core.
- `source/lib*_libretro-v1.0.tar.gz` — complete corresponding source for
  each binary at the commit it was built from. For VICE, the project's
  build-time patches are pre-applied so the tarball is a buildable
  standalone snapshot.
- `SOURCES.md` — upstream pointers, the exact commit each tarball was
  produced from, the VICE patches (with rationale), and reproduce-the-
  build instructions.

## Licensing & source

These cores carry copyleft and weak-copyleft licenses; the binaries you
download are accompanied by the complete corresponding source code in
[`source/`](source) under [GPL §3(a)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.html#section3).
For mGBA (MPL 2.0) the source is included for parity even though MPL's
source-availability requirements are file-level rather than whole-program.

See [SOURCES.md](SOURCES.md) for upstream URLs, exact commits, and the
VICE patch set.

## Building

These binaries are produced from the
[RetroGameDev VR main repo](https://github.com/retrogamedev/retrogamedevvr)'s
`LibretroCores/Makefile`. To reproduce a binary from a source tarball in
this repository, extract the tarball and follow the build commands in
[SOURCES.md](SOURCES.md) for that core.

## Issues

- Bugs in the emulators themselves are best reported to each upstream
  project (see the **Upstream** column above).
- Bugs in how RetroGameDev VR uses these cores belong on the main app
  repository.
