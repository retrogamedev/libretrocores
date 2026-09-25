# RetroGameDev VR — Libretro Cores

This repository hosts the [libretro](https://www.libretro.com/) emulator cores
downloaded at runtime by **RetroGameDev VR** on Meta Quest. The app fetches
`manifest.json` on launch and pulls each core from this repository the first
time a user plays a ROM that needs it.

Each core is versioned independently; see the `version` field per core in
`manifest.json` for the current release. When a core is revised, its source
archive is republished under a new `-vN.N` filename matching that version.

## Cores included

| File | Systems | License | Upstream |
| --- | --- | --- | --- |
| `libmgba_libretro.so` | Game Boy, Game Boy Color, Game Boy Advance | MPL 2.0 | [mgba-emu/mgba](https://github.com/mgba-emu/mgba) |
| `libfceumm_libretro.so` | NES | GPL v2 | [libretro/libretro-fceumm](https://github.com/libretro/libretro-fceumm) |
| `libvice_x64_libretro.so` | Commodore 64 | GPL v2 or later | [libretro/vice-libretro](https://github.com/libretro/vice-libretro) |
| `libgearsystem_libretro.so` | Sega Master System, Game Gear | GPL v3 | [drhelius/Gearsystem](https://github.com/drhelius/Gearsystem) |
| `libbsnes_mercury_balanced_libretro.so` | Super Nintendo (SNES) | GPL v3 | [libretro/bsnes-mercury](https://github.com/libretro/bsnes-mercury) |
| `libmupen64plus_next_libretro.so` | Nintendo 64 | GPL v3 | [libretro/mupen64plus-libretro-nx](https://github.com/libretro/mupen64plus-libretro-nx) |
| `libpcsx_rearmed_libretro.so` | PlayStation (PS1) | GPL v2 | [libretro/pcsx_rearmed](https://github.com/libretro/pcsx_rearmed) |
| `libares_md_libretro.so` | Sega Genesis / Mega Drive | ISC | [ares-emulator/ares](https://github.com/ares-emulator/ares) |
| `libflycast_libretro.so` | Sega Dreamcast | GPL v2 | [flyinghead/flycast](https://github.com/flyinghead/flycast) |
| `libstella_libretro.so` | Atari 2600 | GPL v2 | [stella-emu/stella](https://github.com/stella-emu/stella) |
| `libmame_libretro.so` (Release asset, see below) | Arcade (curated MAME driver set) | GPL v2 + BSD-3 | [libretro/mame](https://github.com/libretro/mame) |

All binaries are built for Android **arm64-v8a** (Meta Quest 3 / 3S / Pro). They
are not portable to other platforms.

## Repository layout

- `manifest.json` — catalogue read by the app's `CoreInstaller`. Contains
  per-core version, download URL, SHA-256, size, source URL, and license
  URL. See [SOURCES.md](SOURCES.md) for the schema.
- `lib*_libretro.so` — the core binaries.
- `LICENSE-mgba.txt`, `LICENSE-fceumm.txt`, `LICENSE-vice.txt`,
  `LICENSE-gearsystem.txt`, `LICENSE-bsnes.txt`, `LICENSE-mupen64.txt`,
  `LICENSE-pcsx.txt`, `LICENSE-ares.txt`, `LICENSE-flycast.txt`,
  `LICENSE-stella.txt`, `LICENSE-mame.txt` — full license text for each core.
- `source/lib*_libretro-vN.N.tar.gz` — complete corresponding source for
  each binary at the commit it was built from. Where a core carries the
  project's build-time patches, they are pre-applied so the tarball is a
  buildable standalone snapshot matching the binary (see SOURCES.md per core).
- `SOURCES.md` — upstream pointers, the exact commit each tarball was
  produced from, the VICE patches (with rationale), and reproduce-the-
  build instructions.
- **GitHub Releases** — large cores live as Release assets rather than in
  git history. MAME (`mame-vN.N` releases) ships four assets: the gzipped
  binary `libmame_libretro.so.gz`, its supported-set list `mame-sets.json`,
  the Lua plugin pack `mame-plugins.zip` (hiscore), and the source archive
  `libmame_libretro-vN.N.tar.gz`. `manifest.json` points at them;
  `LICENSE-mame.txt` stays in the repository.

## Licensing & source

These cores carry copyleft and weak-copyleft licenses; the binaries you
download are accompanied by the complete corresponding source code in
[`source/`](source) under [GPL §3(a)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.html#section3).
For mGBA (MPL 2.0) the source is included for parity even though MPL's
source-availability requirements are file-level rather than whole-program.

See [SOURCES.md](SOURCES.md) for upstream URLs, exact commits, and the
VICE, Mupen64Plus-Next, PCSX-ReARMed and Gearsystem patch sets.

## Building

These binaries are produced from the RetroGameDev VR main
repository's `LibretroCores/Makefile`. To reproduce a binary from a source tarball in
this repository, extract the tarball and follow the build commands in
[SOURCES.md](SOURCES.md) for that core.

## Issues

- Bugs in the emulators themselves are best reported to each upstream
  project (see the **Upstream** column above).
- Bugs in how RetroGameDev VR uses these cores belong on the main app
  repository.
