# Kut Engine: id5-level upgrade plan

Goal: bring Kut Engine (RBDOOM-3-BFG base, Vulkan-only) up to id Tech 5-era technology,
while every feature can scale down to **PS3/Xbox 360-era budgets**.

Kut Engine targets PC only. "PS3 compatible" here means the engine fits within PS3-class
limits when run with the `ps3` budget profile. It does not mean a PlayStation 3 build: no
console SDK or homebrew toolchain is used.

## 1. Budget profiles

Every system reads its limits from a profile chosen with the `kut_profile` cvar (`pc` or `ps3`).
The `ps3` numbers are design targets, roughly the PS3's 256 MB system RAM + 256 MB video RAM.

| Budget | `ps3` profile | `pc` profile |
|---|---|---|
| CPU heap (game + engine) | 200 MB | unlimited, tracked |
| GPU memory, all resources | 200 MB | unlimited, tracked |
| Virtual texture physical cache | 4096×4096 texels, ~32 MB | 8192×8192 or more |
| VT page size | 128×128 (120 + 4 texel border each side) | same |
| Page transcodes per frame | 16 | 64+ |
| Render resolution | 1280×720, adaptive down to ~960×720 | native, adaptive optional |
| Frame target | 30 Hz (33.3 ms) minimum, 60 Hz goal | 60 Hz+ |
| Worker threads | 6 (one per Cell SPU) | all cores |
| Streaming bandwidth assumed | 20 MB/s (HDD/Blu-ray class) | SSD |
| Heavy features | PBR on, shadow atlas reduced, SSGI off | all on |

**Job rule, from the SPU heritage:** a job reads its inputs, writes its outputs and does
nothing else. No following pointers into shared engine state, and working data should fit in
~256 KB (the size of an SPU's local memory). Jobs written this way stay fast on any core count.

## 2. Phases

Each phase ends with something visible running in the engine.

### Phase 0: Foundation
- Rename the executable and game identity: `APP_NAME` in `neo/CMakeLists.txt`, `neo/framework/Licensee.h`.
- Add the `kut_profile` cvar and memory/GPU budget tracking, shown in an ImGui overlay.
- Build a small test map from freely licensed assets in TrenchBroom, so rendering can be checked on screen.
- Verify that the existing adaptive resolution (`neo/renderer/ResolutionScale.cpp`, `rs_enable`)
  works on the NVRHI/Vulkan backend.

### Phase 1: Virtual texturing runtime
New module `neo/renderer/VirtualTexture/`:
- **Page table:** an indirection texture that maps virtual pages to slots in the physical cache, one mip chain.
- **Physical cache:** GPU textures holding resident pages (diffuse+alpha, normal, specular/roughness layers).
- **Feedback pass:** render a low-resolution buffer (e.g. 1/8 screen) of page IDs and mip levels, then read it back on the CPU.
- **Page requests:** dedupe and prioritize requests, evict least recently used pages, fall back to
  coarser mips while a page loads.
- **Transcoding jobs:** decode pages and encode them to BC on worker threads, using
  `neo/renderer/DXT/DXTEncoder_SSE2.cpp`.
- **Shaders:** a VT sampling function in `neo/shaders/` and a `virtualTexture` keyword in materials
  (`neo/renderer/Material.cpp`).
- Done when a test map renders with all its surfaces sampled from one virtual texture, within the `ps3` cache budget.

### Phase 2: VT build tools
- Offline builder: pack the source textures referenced by a map into a virtual texture file (`.kvt`, our own format).
- On-disk format: index + compressed page blobs. Start with BC + deflate, then move to a
  denser codec that is transcoded to BC at runtime once the streaming budget requires it.
- Unique UV layout: generate one atlas over all world surfaces, so each texel is used only once.

### Phase 3: Unique-texel workflow
- Stamping and decal painting tools that bake directly into virtual texture pages.
- Editor preview through the ImGui tools.

### Phase 4: Baked lighting in the virtual texture
- Bake static lighting into VT pages: RBDOOM's irradiance volumes plus direct static lights.
- Dynamic lights and shadows stay real-time on top of the baked lighting.

### Phase 5: Streaming
- Asynchronous level and asset streaming on the file system (`neo/framework/FileSystem.cpp`), held to the profile's bandwidth budget.
- Priority shared with VT page requests, so textures don't starve geometry loads, or the other way round.

### Phase 6: Scale and polish
- Reduced-cost render paths for the `ps3` profile: shadow atlas size, SSAO/SSGI, post-processing.
- Profiling passes against the budgets in section 1.

## 3. References
- J.M.P. van Waveren, "id Tech 5 Challenges: From Texture Virtualization to Massive Parallelization" (SIGGRAPH 2009)
- J.M.P. van Waveren, "Real-Time Texture Streaming & Decompression" and "Real-Time DXT Compression"
- Sean Barrett, "Sparse Virtual Textures" (GDC 2008)
- `docs/DOOM-3-BFG-Technical-Note.pdf` (in this repo): how BFG was restructured for consoles

## 4. Rules
- No assets or code from commercial games. Techniques come from public papers and outside observation only.
- Changed GPL files get a "Modified 2026 for Kut Engine" note in their header.
