# Source code references

This document pins the upstream source for each core in this repository and
documents any local patches applied during the build. It satisfies
"information you received as to the offer to distribute corresponding source
code" as required by GPL §3 — until pinned source archives are uploaded
directly to this repository (planned before Store submission).

## mGBA — `libmgba_libretro.so`

- **Upstream:** https://github.com/mgba-emu/mgba
- **License:** Mozilla Public License 2.0 — see [LICENSE-mgba.txt](LICENSE-mgba.txt).
- **Local patches:** none. Built unmodified.
- **Build configuration:**
  ```
  cmake -DBUILD_LIBRETRO=ON \
        -DBUILD_SHARED_LIBS=ON \
        -DBUILD_SDL=OFF \
        -DBUILD_QT=OFF \
        -DCMAKE_BUILD_TYPE=Release \
        -DUSE_SQLITE3=OFF \
        -DUSE_EDITLINE=OFF \
        -DUSE_MINIZIP=OFF \
        -DCMAKE_SHARED_LINKER_FLAGS="-Wl,-z,max-page-size=16384"
  cmake --build .
  llvm-strip --strip-unneeded mgba_libretro.so
  ```

## FCEUmm — `libfceumm_libretro.so`

- **Upstream:** https://github.com/libretro/libretro-fceumm
- **License:** GNU General Public License version 2 — see [LICENSE-fceumm.txt](LICENSE-fceumm.txt).
- **Local patches:** none. Built unmodified.
- **Build configuration:**
  ```
  ndk-build APP_ABI=arm64-v8a \
            APP_PLATFORM=android-24 \
            APP_LDFLAGS="-Wl,-z,max-page-size=16384"
  ```
  (ndk-build strips for release automatically.)

## VICE x64 — `libvice_x64_libretro.so`

- **Upstream:** https://github.com/libretro/vice-libretro
- **License:** GNU General Public License version 2 or later — see [LICENSE-vice.txt](LICENSE-vice.txt).
- **Build configuration:**
  ```
  ndk-build EMUTYPE=x64 \
            APP_ABI=arm64-v8a \
            APP_PLATFORM=android-24 \
            APP_LDFLAGS="-Wl,-z,max-page-size=16384"
  ```
- **Local patches** (applied at build time, restored after — the upstream
  repo is not modified across a build):

  1. `vice/src/residfp/sidcxx11.h` — disable the auto_ptr compatibility
     block. The Unity NDK's clang defaults to C++17, where `std::auto_ptr`
     has been removed. Upstream's `#ifndef HAVE_CXX11` block tries to alias
     `unique_ptr` to `auto_ptr` when `HAVE_CXX11` is undefined (which it is
     in the libretro build path, where autoconf doesn't run).

  2. `vice/src/residfp/builders/residfp-builder/residfp/FilterModelConfig.h`
     — force the `unique_ptr::deleter_type` branch by replacing
     `#ifdef HAVE_CXX11` with `#if 1`.

  3. `vice/src/residfp/builders/residfp-builder/residfp/FilterModelConfig8580.h`
     — same patch as (2).

  4. `libretro/libretro-core.c` — three patches for clean reload behaviour:
     - Reset `runstate` to `RUNSTATE_FIRST_START` at the end of
       `retro_deinit` so the next `retro_load_game` takes the clean
       first-boot path.
     - Guard `pre_main()` with a static flag so the second call (on reload)
       is a no-op (`machine_shutdown()` is `#if 0`'d in `retro_deinit` so
       the emulator is still initialised).
     - Clear keyboard matrix and joystick state in `reload_restart()`
       before autostart setup, so stale key-down state from the previous
       session doesn't corrupt the BASIC prompt that autostart monitors.

  All four patches are applied at the start of the VICE build target in
  the main repo's `LibretroCores/Makefile` and reverted on success. For
  any future source archive shipped from this repo, the patches will be
  applied to the source rather than left as separate files.

## Manifest schema (`manifest.json`)

```json
{
  "manifestVersion": 1,
  "cores": [
    {
      "id": "<short id, e.g. mgba>",
      "displayName": "<UI label>",
      "system": "<system code, e.g. nes / c64 / gb-gbc-gba>",
      "fileName": "<.so filename>",
      "version": "<YYYY.MM.DD>",
      "downloadUrl": "<raw URL to the .so>",
      "sourceUrl": "<URL to the source archive matching this binary>",
      "licenseUrl": "<URL to the license text>",
      "sha256": "<lowercase hex sha256 of the .so>",
      "sizeBytes": <integer byte count>
    }
  ]
}
```

## TODO before Meta Store submission

- Upload pinned source archives (matching the exact build commit, with the
  VICE patches applied) to this repo or its GitHub Releases.
- Replace the `sourceUrl` and `licenseUrl` placeholders in `manifest.json`
  with real URLs pointing at those archives.
- Lawyer consult on GPL §3 specifics — written-offer wording, retention
  period, third-party redistribution.
