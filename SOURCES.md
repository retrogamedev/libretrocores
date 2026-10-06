# Source code references

This document describes the source for each binary in this repository.
Every binary has its complete corresponding source code shipped alongside
it in [`source/`](source), in line with
[GPL §3(a)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.html#section3).

Binaries and source archives are version **1.0** unless a core's entry below
states otherwise (revised cores carry a higher version and a matching `-vN.N`
archive filename).

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
- **Local patches:** one RGDVR patch, pre-applied in this archive. It exposes
  the SegaScope 3D glasses eye bit to the RetroGameDev VR host so SegaScope
  titles (Space Harrier 3-D, Out Run 3-D, etc.) can render in true per-eye
  stereo 3D on Quest. The patch adds a public `GetGlassesRegistry()` accessor on
  `GearsystemCore` (forwarding `Input`'s glasses register) and, in
  `platforms/libretro/libretro.cpp`, registers a getter with the frontend during
  `retro_init` via a private environment command (`RETRO_ENVIRONMENT_PRIVATE |
  6`). Any frontend that does not recognise that command returns false and the
  registration is ignored, so the core still builds and runs as stock Gearsystem
  everywhere else. No emulation behaviour changes.
- **Reproduce the build** (from the extracted source root; the patch above is
  already applied in this archive):
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

## bsnes-mercury balanced — `libbsnes_mercury_balanced_libretro.so`

- **Upstream:** https://github.com/libretro/bsnes-mercury
- **License:** GNU General Public License version 3 — see
  [LICENSE-bsnes.txt](LICENSE-bsnes.txt).
- **Source commit:** [`ac0b6b1`](https://github.com/libretro/bsnes-mercury/commit/ac0b6b1)
- **Source archive:**
  [source/libbsnes_mercury_balanced_libretro-v1.1.tar.gz](source/libbsnes_mercury_balanced_libretro-v1.1.tar.gz)
- **Local patches:** **already applied in the source archive.** One patch;
  the tarball is a buildable standalone snapshot matching the shipped binary.
  Search the source for `RGDVR` to locate it. Summarised below for transparency.

  1. `sfc/system/video.cpp` — makes `Video::draw_cursor()` an early return, so
     the emulator's own on-screen light-gun crosshair (red for the Super Scope,
     blue for the Justifier) is not drawn. The app renders its own VR aiming
     reticle, so the core-drawn one is a redundant second crosshair over the
     game. One-line change; the core is otherwise stock and still builds and
     runs identically.

- **Profile:** built with `PROFILE=balanced` (scanline-precision PPU). The
  jni `Android.mk` defaults to `performance`, so the profile must be passed
  explicitly or you'll get the wrong core.
- **Reproduce the build** (from the extracted source root):
  ```
  ndk-build -C target-libretro/jni \
            NDK_PROJECT_PATH=target-libretro \
            APP_ABI=arm64-v8a \
            APP_PLATFORM=android-24 \
            APP_LDFLAGS="-Wl,-z,max-page-size=16384" \
            PROFILE=balanced
  ```
  (ndk-build strips for release automatically. Output is
  `target-libretro/libs/arm64-v8a/libretro.so`, renamed to
  `libbsnes_mercury_balanced_libretro.so`.)

## Mupen64Plus-Next + GLideN64: `libmupen64plus_next_libretro.so`

- **Upstream:** https://github.com/libretro/mupen64plus-libretro-nx
  (cloned with submodules).
- **License:** GNU General Public License version 3, see
  [LICENSE-mupen64.txt](LICENSE-mupen64.txt). The mupen64plus core itself is
  GPLv2, but the libretro-nx bundle links additional GPLv3 plugins, so the
  combined binary is GPLv3.
- **Source commit:** [`98c1b0d`](https://github.com/libretro/mupen64plus-libretro-nx/commit/98c1b0d877542b01314b3b04272282ba223b65b3)
- **Source archive:**
  [source/libmupen64plus_next_libretro-v1.0.tar.gz](source/libmupen64plus_next_libretro-v1.0.tar.gz)
- **Renderer:** GLideN64 (OpenGL ES). The Vulkan ParaLLEl-RDP renderer and the
  ParaLLEl (LLE) RSP are disabled for the arm64 build (see patch 4 below), so the
  N64 path is GLideN64 + HLE RSP only. Earlier releases used ParaLLEl-RDP; moving
  to GLideN64 removed the ~20 MB Vulkan renderer and its static-global teardown
  hazards.
- **Local patches:** already applied in the source archive. The tarball is a
  buildable standalone snapshot matching the shipped binary. Search the source
  for `RGDVR` to locate the inline changes. Summarised for transparency:

  1. `libretro/libretro.c`: appends a `retro_swap_rom()` entry point for
     in-process N64 to N64 content switching, and adds an `"auto"` case to the
     controller-pak selection that picks Memory Pak vs Rumble Pak per game from
     the ROM settings (savetype None + mempak gives Memory Pak, else Rumble if
     the game supports it).

  2. `libretro-common/libco/aarch64.c`: full-file replacement adding the
     callee-saved FP registers `d8` to `d15` to `co_switch_aarch64`. Upstream
     saves `x19` to `x29` but not `d8` to `d15` (which AAPCS64 requires preserved
     across a call), so a `double` / NEON local held across `retro_run()`'s
     co-switch is corrupted on return.

  3. `GLideN64/src/FrameBuffer.cpp`: re-enables the `clearColorBuffer` at the end
     of `FrameBuffer::init` (binding the FBO first). Upstream left it commented
     out, so a freshly allocated framebuffer colour texture held uninitialised
     VRAM; a game that samples a framebuffer for an effect on the same frame it
     is first created (Ocarina of Time title-logo shimmer) showed one frame of
     random-coloured garbage. ParaLLEl-RDP cleared its buffers, so this was a
     GLideN64-only regression.

  4. `libretro/jni/Android.mk`: sets `HAVE_PARALLEL_RSP = 0` and
     `HAVE_PARALLEL_RDP = 0` for the arm64-v8a build, dropping the Vulkan
     ParaLLEl-RDP renderer and the ParaLLEl LLE RSP from the core (GLideN64 +
     HLE only).

  5. `mupen64plus-core/src/plugin/plugin.c`: falls back to the GLideN64 graphics
     plugin if `plugin_connect_all()` would otherwise leave the global `gfx`
     struct unassigned. The function selects the RDP plugin with a switch on
     `current_rdp_type`; the angrylion and ParaLLEl cases are compiled out by
     patch 4 above, and `RDP_PLUGIN_NONE` / `default` fall through, so those
     paths left `gfx` as zeroed BSS. Nothing null-checks it, and `main_run()`
     then calls `gfx.romOpen()` on a NULL pointer, crashing with a jump to
     address 0. GLideN64 is the only renderer compiled into this build, so
     defaulting to it is both safe and correct.

  A further inline change to `mupen64plus-video-paraLLEl/rdp.cpp` (a re-init
  teardown-order fix) remains in the source tree for completeness but is no
  longer compiled, since ParaLLEl-RDP is disabled by patch 4 above.

- **Reproduce the build** (from the extracted source root):
  ```
  ndk-build -C libretro/jni \
            NDK_PROJECT_PATH=libretro \
            APP_ABI=arm64-v8a \
            APP_PLATFORM=android-24 \
            APP_LDFLAGS="-Wl,-z,max-page-size=16384"
  ```
  (ndk-build strips for release automatically. Output is
  `libretro/libs/arm64-v8a/libretro.so`, renamed to
  `libmupen64plus_next_libretro.so`.)


## PCSX-ReARMed — `libpcsx_rearmed_libretro.so`

- **Upstream:** https://github.com/libretro/pcsx_rearmed
  (cloned with submodules — `frontend/libpicofe`).
- **License:** GNU General Public License version 2 — see
  [LICENSE-pcsx.txt](LICENSE-pcsx.txt).
- **Source commit:** [`d26eaee5`](https://github.com/libretro/pcsx_rearmed/commit/d26eaee5c8fb47c1832b8bf32c1358d625da8a02)
- **Source archive:**
  [source/libpcsx_rearmed_libretro-v1.3.tar.gz](source/libpcsx_rearmed_libretro-v1.3.tar.gz)
- **Local patches:** **already applied in the source archive.** Five patches;
  the tarball is a buildable standalone snapshot matching the shipped binary.
  Search the source for `RGDVR` to locate them.

  1. `libpcsxcore/pad.c` — auto-enable DualShock analog mode for all games.
     The emulated DualShock boots in digital mode and PCSX only auto-switches
     to analog for games that probe the controller config protocol (many,
     including Gran Turismo 2, never do), leaving the analog sticks dead. The
     patch drops the `configModeUsed` gate on PCSX's own auto-analog path so
     every game switches to analog ~16 polls after boot. One-line change to the
     auto-analog condition, anchored on `CMD_READ_DATA_AND_VIBRATE` so it only
     touches the auto-analog check, not the identical idle-timeout check above it.

  2. `frontend/libretro-version-script` — export the `CdromId` symbol. The
     linker version-script restricts the dynamic symbol table to `retro_*`,
     hiding everything else, so the host's `dlsym("CdromId")` fails. The patch
     adds `CdromId` to the `global:` list so the host can read the loaded
     disc's serial (SLES/SLUS/SCES/...) for per-game light-gun configuration.
     `CdromId` is referenced internally so it survives `--gc-sections`; only
     its export visibility changes, no emulation behaviour is affected.

  3. `libpcsxcore/psxmem.h` — HLE BIOS scratchpad pointer fix. Upstream commit
     `dd2225ab` made `psxm()` return a BIOS-ROM (`psxR`) pointer for scratchpad
     addresses (0x1f800000..0x1f8003ff); the correct buffer is `psxH`
     (scratchpad plus hardware registers, see `r3000a.h`). `psxm()` backs the
     HLE BIOS `Ra0`/`Ra1` argument helpers, so any HLE call whose pointer
     arguments live in scratchpad read zeros and wrote into the wrong buffer.
     One-line change (`psxR` to `psxH`); new in the v1.2 build.

  4. `libpcsxcore/psxbios.c` — deliver I/O-complete events from the HLE
     `delete()` file call. On a real BIOS, `delete("bu00:...")` performs card
     I/O and raises the completion events (HwCARD `f4000001` spec 4, SwCARD
     `f0000011` spec 4). The HLE version finishes instantly and delivered
     nothing, so games whose memory-card state machine polls those events
     after every operation spin forever (Azure Dreams hangs after deleting
     its write-probe file). The patch raises both events after each card
     delete, mirroring the existing upstream `firstfile()` precedent; new in
     the v1.2 build.

  5. `libpcsxcore/psxbios.c` — clear the memory card's "new card" flag after
     `firstfile()`. On a real BIOS, reading the card directory (`bu_init`)
     checks the "MC" header and then test-writes sector 0x3F, and it is that
     write which clears the card's new-card flag, so the next `_card_info`
     reports 4 (ok). The HLE `firstfile()` only scans the in-memory card and
     left the flag set, so `_card_info` kept answering 2000 (new card) until
     an actual write. Games that wait for 4 before writing (Xenogears loops
     `_card_info`/`firstfile` at every save point and reports the card as
     unformatted) never got there. Two one-line additions after the
     `bufile()` calls in `firstfile()`, one per card; new in the v1.3 build.

- **Reproduce the build** (from the extracted source root):
  ```
  ndk-build -C jni \
            NDK_PROJECT_PATH=. \
            APP_ABI=arm64-v8a \
            APP_PLATFORM=android-24 \
            APP_LDFLAGS="-Wl,-z,max-page-size=16384"
  ```
  (ndk-build strips for release automatically. The arm64-v8a build pulls in the
  native MIPS dynarec + libchdr. Output is `libs/arm64-v8a/libretro.so`, renamed
  to `libpcsx_rearmed_libretro.so`.)

## ares (Mega Drive / Genesis) — `libares_md_libretro.so`

- **Upstream:** https://github.com/ares-emulator/ares
- **License:** ISC — see [LICENSE-ares.txt](LICENSE-ares.txt). (That file also
  bundles the licenses of ares' vendored third-party libraries.)
- **Source commit:** [`449b937`](https://github.com/ares-emulator/ares/commit/449b937)
- **Version:** 1.1 (see change notes below).
- **Source archive:**
  [source/libares_md_libretro-v1.1.tar.gz](source/libares_md_libretro-v1.1.tar.gz)
- **Local patches:** **already applied in the source archive.** The tarball is a
  buildable standalone snapshot matching the shipped binary. ares ships no
  libretro target of its own, so a thin libretro shim is bundled in-tree at
  `ares-libretro/` and wired in through ares' top-level `CMakeLists.txt` in place
  of the desktop frontend, with the build trimmed to the Mega Drive core and
  pointed at ares' Linux toolchain config for the Android/arm64 cross-build. The
  one cartridge-level change exposes the active battery save-RAM buffer to the
  host — `saveData()`/`saveSize()` on the MD board interface plus the `linear`
  and `standard` boards, and a `#pragma once` added to `nall/nall/decode/mmi.hpp`
  — so battery saves can be read back and restored. Two light-gun peripherals
  (Konami Justifier, Sega Menacer) are added under `ares/md/controller/` and wired
  into the controller port, sharing a small VDP hook that registers the aimed
  pixel and, when the raster beam crosses it, raises the external interrupt and
  re-latches the HV counter (so both live-read and M3 HV-latch games read the aim).
  The shim also derives the console TV standard (NTSC/PAL) from the ROM header
  region field instead of forcing NTSC.
- **Change notes:**
  - **1.1** — Menacer button/detection fix (buttons gated to the read strobe so the
    peripheral ID stays valid while a button is held; fixes T2: The Arcade Game menu
    detection), light-gun HV-counter re-latch on the light pulse (fixes Menacer aim
    position), and header-derived region so PAL-only carts (e.g. Body Count) run at
    50 Hz instead of locking out.
  - **1.0** — Initial release.
- **Reproduce the build** (from the extracted source root):
  ```
  # 1) Build ares' `sourcery` resource compiler natively -- it is a host codegen
  #    tool that runs on the build machine, not the Quest. (On macOS append
  #    `-framework Foundation -framework Cocoa`.)
  clang++ -std=c++20 -O2 -DNALL_HEADER_ONLY -I. -Inall \
          tools/sourcery/sourcery.cpp -o build_native/sourcery
  #    then write build_native/sourceryConfig.cmake importing that binary as an
  #    IMPORTED `sourcery` target so ares' cross build finds it via find_package.

  # 2) Cross-compile the Mega Drive core for Android arm64:
  cmake -B build \
        -DCMAKE_TOOLCHAIN_FILE=<Android NDK toolchain.cmake> \
        -DANDROID_ABI=arm64-v8a \
        -DANDROID_PLATFORM=android-24 \
        -DARES_CROSSCOMPILING=ON \
        -DARES_ENABLE_CHD=OFF \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_SHARED_LINKER_FLAGS="-Wl,-z,max-page-size=16384"
  cmake --build build --target ares_md_libretro -j
  llvm-strip --strip-unneeded <build>/ares-libretro/libares_md_libretro.so
  ```
  (Output is renamed to `libares_md_libretro.so`.)

## Flycast (Dreamcast) — `libflycast_libretro.so`

- **Upstream:** https://github.com/flyinghead/flycast
- **License:** GPLv2 — see [LICENSE-flycast.txt](LICENSE-flycast.txt).
- **Source commit:** [`e9b7fb651`](https://github.com/flyinghead/flycast/commit/e9b7fb651) (`v2.6-330-ge9b7fb651`)
- **Source archive:**
  [source/libflycast_libretro-v1.1.tar.gz](source/libflycast_libretro-v1.1.tar.gz)
  — Flycast's own source only. Its dependencies are large third-party **git
  submodules** (SDL, glslang, Vulkan-Headers, VulkanMemoryAllocator, oboe, libchdr,
  luabridge, libadrenotools, rcheevos, ..., all listed in `.gitmodules`, ~527M),
  each publicly available and pinned by the upstream commit below. They're excluded
  from the archive (they'd exceed GitHub's per-file size limit); fetch them with:
  ```
  git clone --recurse-submodules https://github.com/flyinghead/flycast
  cd flycast && git checkout e9b7fb651 && git submodule update --init --recursive
  ```
  then overlay this archive's patched sources (or just apply the patch below).
- **Local patches:** **already applied in the source archive.** Two patches, both in
  `core/hw/maple/maple_devs.cpp`; neither changes renderer or emulation behaviour:
  1. `maple_sega_vmu::OnSetup`: the VMU is only reformatted when there was genuinely
     no existing card file, guarding the change `if (sum == 0)` →
     `if (sum == 0 && rfile == nullptr)`. Without it, a maple device reconnect (e.g. a
     controller/light-gun device change) that momentarily re-reads an existing save as
     empty would reformat and blank it.
  2. VMU write durability (new in v1.1): each VMU block write and `fullSave` is now
     followed by `fflush` (folded into the existing write error check). Flycast writes
     the memory card through a buffered stdio handle, so on an abrupt teardown the last
     buffered write — typically the final phase of the FAT block — could be lost,
     leaving a save whose data and directory reached disk but whose FAT chain was
     truncated at a 128-byte boundary; the game then reads it back as "corrupted".
     Flushing each write makes it durable immediately.
- **Reproduce the build** (from the extracted source root):
  ```
  cmake -S . -B build \
        -DCMAKE_TOOLCHAIN_FILE=<Android NDK toolchain.cmake> \
        -DANDROID_ABI=arm64-v8a \
        -DANDROID_PLATFORM=android-24 \
        -DLIBRETRO=ON \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_SHARED_LINKER_FLAGS="-Wl,-z,max-page-size=16384 -Wl,--no-as-needed -landroid -Wl,--as-needed"
  cmake --build build -j
  llvm-strip --strip-unneeded <build>/.../flycast_libretro.so
  ```
  (Output is renamed to `libflycast_libretro.so`. The `-landroid` link is required:
  without it `ASharedMemory_create` is null at runtime and guest RAM allocation
  fails, crashing after the Sega logo.)

## Stella (Atari 2600) — `libstella_libretro.so`

- **Upstream:** https://github.com/stella-emu/stella (mainline Stella; its
  libretro port lives in-tree under `src/os/libretro`). This is NOT the old
  `libretro/stella2014-libretro` fork.
- **License:** GNU General Public License version 2 — see
  [LICENSE-stella.txt](LICENSE-stella.txt).
- **Source commit:** [`c65c845c8`](https://github.com/stella-emu/stella/commit/c65c845c8)
  (`7.0-869-gc65c845c8`, master, 2026-09-10; untagged because the libretro
  port's controller autodetect and modern bankswitching landed after the 7.0 tag).
- **Source archive:**
  [source/libstella_libretro-v1.0.tar.gz](source/libstella_libretro-v1.0.tar.gz)
- **Local patches:** **already applied in the source archive.** Two small
  toolchain-compatibility patches (the Unity NDK r27 ships clang 18 / libc++ 18,
  which predate two C++20/23 library and language features Stella uses) plus
  one paddle-input fix. Search the source for `RGDVR:` to locate them.
  1. `src/common/Variant.hxx` — libc++ 18 has no floating-point
     `std::from_chars` (it arrived in LLVM 20). The float and double
     string-parse branches use `std::strtof` / `std::strtod` on a
     null-terminated copy instead. Integer `from_chars` elsewhere is untouched.
  2. `src/common/StellaKeys.hxx`, `src/emucore/tia/TIA.cxx` —
     `Bitmask::Enum{x}` deduces through an alias template (C++20 alias CTAD),
     which clang supports only from version 19. Rewritten to the class template
     spelling `Bitmask::BitmaskEnum{x}`, which has an explicit deduction guide
     and is already used elsewhere in the same file.
  3. `src/emucore/Paddles.cxx` — paddle position sync. Stella keeps the
     analog-axis paddle position and the digital/mouse position (`myPosition`)
     separately, applying the analog one only on frames where the axis value
     changes and the digital one otherwise. With a thumbstick driving the axis
     in relative mode the axis goes static on release, so the pot snapped back
     to the stale digital position. Two inserted lines write the analog
     position into `myPosition` whenever the analog path applies it, so a
     released stick leaves the paddle where it is; two more gate the digital
     (d-pad) paddle events on the analog axis being at rest, so a stick held
     still in absolute mode doesn't creep the paddle. No effect when no analog
     axis is in use.
- **Reproduce the build** (from the extracted source root):
  ```
  ndk-build -C src/os/libretro/jni \
            NDK_PROJECT_PATH=src/os/libretro \
            APP_ABI=arm64-v8a \
            APP_PLATFORM=android-24 \
            APP_LDFLAGS="-Wl,-z,max-page-size=16384"
  ```
  (ndk-build strips for release automatically. Output `libretro.so` is renamed
  to `libstella_libretro.so`. Requires a C++23-capable NDK clang; NDK r27 / clang
  18 was used.)

## MAME (Arcade) — `libmame_libretro.so`

- **Upstream:** https://github.com/libretro/mame (mainline MAME with its
  libretro OSD in-tree under `src/osd/libretro`; tracks upstream
  [mamedev/mame](https://github.com/mamedev/mame)). NOT one of the
  `mame2003` / `mame2010` / `mame2016` snapshots.
- **License:** GNU General Public License version 2 (MAME as a whole), with
  individual files under BSD-3-Clause and other permissive licenses as noted
  in their headers — see [LICENSE-mame.txt](LICENSE-mame.txt) (MAME's
  `COPYING` plus the GPL-2.0 and BSD-3-Clause texts from `docs/legal`).
- **Source commit:** [`4fc9a931`](https://github.com/libretro/mame/commit/4fc9a931)
  (libretro/mame `master`, 2026-09-04).
- **Distribution:** this core is NOT stored in the repository. The binary is
  60 MB and would bloat git history on every rebuild, so the binary (gzipped),
  the supported-set list and the source archive are attached to the GitHub
  Release tagged `mame-v1.0`; `manifest.json` points at those assets.
  - `libmame_libretro.so.gz` — the core, gzip-compressed. The manifest entry
    carries `"compression": "gzip"`; its `sha256` / `sizeBytes` describe the
    `.gz` and the app inflates after verifying.
  - `mame-sets.json` — the ROM sets this build supports (name, title, parent,
    BIOS / not-working flags and, from format 2, the CRC32s of the files each
    set's zip must hold itself, which the app checks uploads against),
    generated from the build's own driver list by
    `LibretroCores/tools/mame-setlist.py` (`make mame-sets` regenerates it
    without rebuilding the core). Referenced by `setListUrl`. Format 2
    replaced the format 1 asset on this release on 2026-10-06; same core.
  - `mame-plugins.zip`: MAME's own Lua plugin files, taken unmodified from
    the `plugins/` directory of the same source tree: the `hiscore` plugin
    (`init.lua`, `plugin.json`, `sort_hiscore.lua` and `hiscore.dat`), the
    `json` module it requires, and `boot.lua`, the plugin bootstrap. The app
    unpacks it into MAME's plugin path and enables `hiscore`, which gives
    persistent high scores to boards with no battery-backed RAM. Built by
    `make mame-plugins` (reproducible: file mtimes are pinned, so the digest
    changes only when the contents do). Referenced by `pluginsUrl`, verified
    against `pluginsSha256`. Licences, per the file headers: `hiscore/init.lua`
    CC0; `json/init.lua` MIT (its `LICENSE` is included in the zip);
    `boot.lua` BSD-3-Clause; `hiscore.dat` carries no licence header and is
    distributed here exactly as it is in the MAME repository.
  - `libmame_libretro-v1.0.tar.gz` — complete corresponding source.
- **Source archive:** `libmame_libretro-v1.0.tar.gz` on the `mame-v1.0`
  Release (the full `libretro/mame` tree at the commit above, minus `.git`
  and build output; ~216 MB).
- **Local patches:** none. Built unmodified from upstream.
- **Curated driver set.** Not a full arcade build: the binary contains only
  the drivers listed in `MAME_DRIVERS` in `LibretroCores/Makefile` (43 driver
  source files: Namco, Nintendo (incl. Vs. System), Midway 8080, Atari, Williams,
  Taito (incl. Operation Wolf), Konami (incl. Lethal Enforcers, System GX), Namco NB-1 (Point Blank), Capcom incl. CPS1/CPS2, Neo Geo, Sega
  System 1/16/Out Run, Irem, Data East, Tecmo, Technos, Toaplan). `mame-sets.json` is the
  authoritative list.
- **Reproduce the build** (from the extracted source root, macOS host; the
  Unity NDK r27 / clang 18 in `ANDROID_NDK_HOME`):
  ```
  make -j10 REGENIE=1 VERBOSE=1 NOWERROR=1 OSD=retro CONFIG=libretro OPTIMIZE=s \
       NO_USE_MIDI=1 NO_USE_PORTAUDIO=1 PYTHON_EXECUTABLE=python3 \
       TARGETOS=android-arm64 gcc=android-arm64 PLATFORM=arm64 ARCHITECTURE= \
       TARGET=mame SUBTARGET=rgdvr \
       SOURCES=<the MAME_DRIVERS list, comma-separated, as src/mame/<vendor>/<driver>.cpp> \
       LDOPTS="-Wl,-z,max-page-size=16384 -Wl,--version-script=src/osd/libretro/libretro-internal/link.T -Wl,--gc-sections" \
       android-arm64
  llvm-strip --strip-unneeded rgdvr_libretro_android.dylib   # GENie names the Android ELF by host OS
  ```
  Output is renamed to `libmame_libretro.so` and gzipped (`gzip -9 -n`).
  The explicit `android-arm64` goal and `ARCHITECTURE=` work around MAME's
  default goal appending the host compiler suffix on Apple Silicon.

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
      "sha256":      "<lowercase hex sha256 of the downloaded file>",
      "sizeBytes":   <integer byte count of the downloaded file>,
      "compression": "<optional: \"gzip\" when downloadUrl serves the .so gzipped; sha256/sizeBytes then describe the .gz>",
      "setListUrl":  "<optional: URL of the core's supported-set list (MAME: mame-sets.json), installed with the core>"
    }
  ]
}
```
