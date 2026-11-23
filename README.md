# Kut Engine

Kut Engine is an open-source 3D game engine. It builds on
[RBDOOM-3-BFG](https://github.com/RobertBeckebans/RBDOOM-3-BFG), which is derived from the
id Tech 4 source code (DOOM 3 BFG Edition GPL release).

The goal is to extend this base with large-world rendering techniques such as virtual texturing,
unique-texel workflows, baked lighting and adaptive resolution, while keeping RBDOOM-3-BFG's
modern foundation: DX12/Vulkan through NVRHI, PBR shading, baked GI and TrenchBroom support.

> Status: early. Kut Engine is **Vulkan-only**: DX12 is off by default and `r_graphicsAPI` is
> locked to `vulkan`. Otherwise the code still matches upstream RBDOOM-3-BFG.

## Roadmap

1. **Virtual texturing**: feedback pass, page cache, async page streaming, runtime BC transcoding
2. **Unique-texture workflow**: painting and baking tools for per-surface texels
3. **Baked lighting** stored in the virtual texture, alongside dynamic lights
4. **Adaptive resolution** to hold a target frame rate
5. **Async streaming** of levels and assets

## Building

Kut Engine builds the same way as RBDOOM-3-BFG for now. See
[README-RBDOOM.md](README-RBDOOM.md) for Windows, Linux and macOS instructions.

Windows quick start (needs Visual Studio 2022, CMake, the Vulkan SDK, and
[ISPC](https://github.com/ispc/ispc/releases) unpacked to `tools/ispc/bin/ispc.exe`):

```
cmake -S neo -B build -G "Visual Studio 17 2022" -A x64 -DFFMPEG=ON -DBINKDEC=OFF
cmake --build build --config Release --parallel
```

Get the source with submodules:

```
git clone --recursive https://github.com/ASELIMTASALTIN/kut-engine.git
```

## Game data

This repository contains **engine code only**. It does not include, and must never include,
assets from any commercial game. Use your own content or freely licensed assets.

## License

Kut Engine is licensed under the **GNU General Public License v3**, the same license as the code
it derives from. See [LICENSE.md](LICENSE.md) for details and [LICENSE_EXCEPTIONS.md](LICENSE_EXCEPTIONS.md)
for parts covered by other licenses. Third-party libraries under `neo/libs` and `neo/extern` keep
their own licenses.

## Credits

- **RBDOOM-3-BFG** by Robert Beckebans and contributors
- **DOOM 3 BFG Edition GPL source** by id Software

Kut Engine is not affiliated with or endorsed by id Software, ZeniMax Media or Bethesda.
DOOM and id Tech are trademarks of their respective owners.
