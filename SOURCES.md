# Source code references

This document describes the source for each binary in this repository.
Every binary has its complete corresponding source code shipped alongside
it in [`source/`](source), in line with
[GPL §3(a)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.html#section3).

All binaries and source archives are version **1.0**.

## mGBA — `libmgba_libretro.so`

- **Upstream:** https://github.com/mgba-emu/mgba
- **License:** Mozilla Public License 2.0 — see
  [LICENSE-mgba.txt](LICENSE-mgba.txt).
- **Source commit:** [`9a36d6576`](https://github.com/mgba-emu/mgba/commit/9a36d6576)
- **Source archive:**
  [source/libmgba_libretro-v1.0.tar.gz](source/libmgba_libretro-v1.0.tar.gz)
- **Local patches:** none. Built unmodified from upstream.
- **Reproduce the build** (from the extracted source root):
  ```
  cmake -B build \
        -DCMAKE_TOOLCHAIN_FILE=<Android NDK toolchain.cmake> \
        -DANDROID_ABI=arm64-v8a \
        -DANDROID_PLATFORM=android-24 \
        -DBUILD_LIBRETRO=ON \
        -DBUILD_SHARED_LIBS=ON \
        -DBUILD_SDL=OFF \
        -DBUILD_QT=OFF \
        -DCMAKE_BUILD_TYPE=Release \
        -DUSE_SQLITE3=OFF \
        -DUSE_EDITLINE=OFF \
        -DUSE_MINIZIP=OFF \
        -DCMAKE_SHARED_LINKER_FLAGS="-Wl,-z,max-page-size=16384"
  cmake --build build -j
  llvm-strip --strip-unneeded build/mgba_libretro.so
  ```

## FCEUmm — `libfceumm_libretro.so`

- **Upstream:** https://github.com/libretro/libretro-fceumm
- **License:** GNU General Public License version 2 — see
  [LICENSE-fceumm.txt](LICENSE-fceumm.txt).
- **Source commit:** [`3a84a6f`](https://github.com/libretro/libretro-fceumm/commit/3a84a6f)
- **Source archive:**
  [source/libfceumm_libretro-v1.0.tar.gz](source/libfceumm_libretro-v1.0.tar.gz)
- **Local patches:** none. Built unmodified from upstream.
- **Reproduce the build** (from the extracted source root):
  ```
  ndk-build -C jni \
            APP_ABI=arm64-v8a \
            APP_PLATFORM=android-24 \
            APP_LDFLAGS="-Wl,-z,max-page-size=16384"
  ```
  (ndk-build strips for release automatically.)

## VICE x64 — `libvice_x64_libretro.so`

- **Upstream:** https://github.com/libretro/vice-libretro
- **License:** GNU General Public License version 2 or later — see
  [LICENSE-vice.txt](LICENSE-vice.txt).
- **Source commit:** [`b0d88812a0`](https://github.com/libretro/vice-libretro/commit/b0d88812a0)
- **Source archive:**
  [source/libvice_x64_libretro-v1.0.tar.gz](source/libvice_x64_libretro-v1.0.tar.gz)
- **Local patches:** **already applied in the source archive.** Four
  patches affecting four files; the tarball is a buildable standalone
  snapshot matching the shipped binary. Search the source for `RGDVR:`
  to locate each. The patches are summarised below for transparency.

  1. `vice/src/residfp/sidcxx11.h` — disable the `auto_ptr` compatibility
     block. The Android NDK's clang defaults to C++17, where
     `std::auto_ptr` has been removed. Upstream's `#ifndef HAVE_CXX11`
     block tries to alias `unique_ptr` to `auto_ptr` when `HAVE_CXX11`
     is undefined (which it is in the libretro build path, where
     autoconf doesn't run). Replaces the `#ifndef` line with
     `#if 0 /* RGDVR: C++17, skip auto_ptr compat */`.

  2. `vice/src/residfp/builders/residfp-builder/residfp/FilterModelConfig.h`
     — force the `unique_ptr::deleter_type` branch by replacing
     `#ifdef HAVE_CXX11` with `#if 1 /* RGDVR: C++17 */`.

  3. `vice/src/residfp/builders/residfp-builder/residfp/FilterModelConfig8580.h`
     — same one-line change as (2).

  4. `libretro/libretro-core.c` — three clean-reload fixes (each a
     single-line insertion):
     - Reset `runstate` to `RUNSTATE_FIRST_START` at the end of
       `retro_deinit` so the next `retro_load_game` takes the clean
       first-boot path.
     - Guard `pre_main()` with a static flag so the second call (on
       reload) is a no-op (`machine_shutdown()` is `#if 0`'d in
       `retro_deinit` so the emulator is still initialised).
     - Clear keyboard matrix and joystick state in `reload_restart()`
       before autostart setup, so stale key-down state from the
       previous session doesn't corrupt the BASIC prompt that autostart
       monitors.

- **Reproduce the build** (from the extracted source root):
  ```
  ndk-build -C jni \
            EMUTYPE=x64 \
            APP_ABI=arm64-v8a \
            APP_PLATFORM=android-24 \
            APP_LDFLAGS="-Wl,-z,max-page-size=16384"
  ```

## Gearsystem — `libgearsystem_libretro.so`

- **Upstream:** https://github.com/drhelius/Gearsystem
- **License:** GNU General Public License version 3 — see
  [LICENSE-gearsystem.txt](LICENSE-gearsystem.txt).
- **Source commit:** [`018c248`](https://github.com/drhelius/Gearsystem/commit/018c248)
  (Gearsystem 3.9.9, +9 commits)
- **Source archive:**
  [source/libgearsystem_libretro-v1.0.tar.gz](source/libgearsystem_libretro-v1.0.tar.gz)
- **Local patches:** none. Built unmodified from upstream.
- **Reproduce the build** (from the extracted source root):
  ```
  ndk-build -C platforms/libretro/jni \
            NDK_PROJECT_PATH=platforms/libretro \
            APP_ABI=arm64-v8a \
            APP_PLATFORM=android-24 \
            APP_LDFLAGS="-Wl,-z,max-page-size=16384"
  ```
  (ndk-build strips for release automatically. Output is
  `platforms/libretro/libs/arm64-v8a/libretro.so`, renamed to
  `libgearsystem_libretro.so`.)

## Manifest schema (`manifest.json`)

```json
{
  "manifestVersion": 1,
  "cores": [
    {
      "id": "<short id, e.g. mgba>",
      "displayName": "<UI label>",
      "system": "<system code, e.g. nes / c64 / gb/gbc/gba>",
      "fileName": "<.so filename>",
      "version": "<release version string, e.g. 1.0>",
      "downloadUrl": "<URL to the .so>",
      "sourceUrl":   "<URL to the source archive>",
      "licenseUrl":  "<URL to the license text>",
      "sha256":      "<lowercase hex sha256 of the .so>",
      "sizeBytes":   <integer byte count>
    }
  ]
}
```
