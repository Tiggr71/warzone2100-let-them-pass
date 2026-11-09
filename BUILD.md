# Build and verify Let Them Pass

Base revision is [Warzone 2100 4.7.0](https://github.com/Warzone2100/warzone2100/releases/tag/4.7.0), commit `7d3d8e51a61ceee89e6bf672a41337e1a9cd7392`. Extract the modified source ZIP to build directly. To start from upstream instead, check out that tag, initialize recursive submodules, and apply `let-them-pass-4.7.0.patch` with `git apply`.

## Windows x64 build

This bundle was built with MSYS2 UCRT64, GCC 16.2.0, CMake 4.4.5, and Ninja. In a UCRT64 shell with the MSYS2 package database updated, install the build tools and dependencies:

```bash
pacman -S --needed git \
  mingw-w64-ucrt-x86_64-{gcc,cmake,ninja,pkgconf,7zip,curl,freetype,fribidi,gettext-tools,harfbuzz,libjpeg-turbo,libogg,libpng,libsodium,libtheora,libvorbis,libzip,miniupnpc,openal,opus,physfs,sdl3,sqlite}
```

From the source root, configure and compile:

```bash
wz_source="$(cygpath -am .)"
wz_build="$(cygpath -am ../wz-build)"
wz_prefix_maps="-ffile-prefix-map=$wz_source=. -ffile-prefix-map=$wz_build=./build"
cmake -S . -B ../wz-build -G Ninja \
  -DCMAKE_BUILD_TYPE=Release -DCMAKE_POLICY_VERSION_MINIMUM=3.5 \
  "-DCMAKE_C_FLAGS=$wz_prefix_maps" "-DCMAKE_CXX_FLAGS=$wz_prefix_maps" \
  -DZIP_EXECUTABLE="$(cygpath -m /ucrt64/bin/7z.exe)" \
  -DENABLE_DOCS=OFF -DENABLE_NLS=OFF -DWZ_ENABLE_BACKEND_VULKAN=OFF \
  -DWZ_ENABLE_BASIS_UNIVERSAL=OFF -DWZ_BUILD_SENTRY=OFF \
  -DENABLE_GNS_NETWORK_BACKEND=OFF -DWZ_ENABLE_WARNINGS_AS_ERRORS=OFF
cmake --build ../wz-build --target warzone2100
```

Use the resulting `wz-build/src/warzone2100.exe` with the generated `wz-build/data` and the UCRT64 runtime libraries. To run from the UCRT64 shell, use:

```bash
mkdir -p ../wz-test-profile
../wz-build/src/warzone2100.exe \
  --datadir="$(cygpath -am ../wz-build/data)" \
  --configdir="$(cygpath -am ../wz-test-profile)"
```

The release archive includes the required DLLs beside the executable. `msys2-runtime-sources.csv` lists their package versions and source locations. The matching MSYS2 source archives are in `msys2-archives` with SHA-256 checksums. The build uses OpenGL and uncompressed texture assets generated with the build. The package also includes the game's optional cutscene and music data. The compiler compatibility edits in the patch support the native Windows GCC build.

## Engine changes

- `src/move.cpp` detects stalled opposing movement, chooses right of way, tests temporary steering targets, and alternates priority on repeat disputes. It also schedules a deterministic per-droid retry when a Move pause lacks its collision timer.
- `src/movedef.h` stores transient traffic and retry state alongside each droid's movement state. The existing save format supplies the underlying order and waypoints.
- `src/order.cpp` clears the transient state when a new order is accepted.

## Verification

A local campaign replay of the same saved jam compared stock movement with the rule enabled over 180 game seconds. Stock movement left all 62 tracked droids on their starting tiles. With the rule enabled, all 62 moved, 60 advanced at least five tiles toward their goals, and 11 finished their moves within two tiles of the goal. The number of droids in the bottleneck fell from 64 to 3.

In a second local replay, three selected droids held Move orders but had the same positions and waypoints across two saves about 92 game seconds apart. All three were in `MOVEPAUSE` with a cleared bump timer. They resumed at 300, 500, and 800 ms with the retry enabled and moved about 18–22 tiles over 20 game seconds with their original orders.

To exercise the rule on another map, issue opposing Move orders through a tight passage in a single-player campaign. Let the units run without reissuing orders and check whether a recurring stall swaps priority and the droids resume their original routes. To verify pause recovery, load a compatible campaign save that already has a ground droid with `MOVEPAUSE`, an active Move order, and a zero bump timer.
