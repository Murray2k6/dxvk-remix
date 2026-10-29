# Remix Plus

[![Build Status](https://github.com/RemixProjGroup/dxvk-remix/actions/workflows/build.yml/badge.svg)](https://github.com/RemixProjGroup/dxvk-remix/actions/workflows/build.yml)

**Remix Plus** is a community fork of NVIDIA RTX Remix
([`dxvk-remix`](https://github.com/NVIDIAGameWorks/dxvk-remix)), the runtime that path-traces classic games.
It is created and led by [Kim2091](https://github.com/Kim2091).

> Remix Plus is not affiliated with, endorsed by or supported by NVIDIA. NVIDIA, RTX and DLSS are trademarks of
> NVIDIA Corporation. Report Remix Plus bugs in this repository's issue tracker, not to NVIDIA or to DXVK.

Like upstream `dxvk-remix`, Remix Plus descends from [DXVK](https://github.com/doitsujin/dxvk). It is a hard fork:
it does not rebase onto upstream, and ports upstream features one at a time.

## Features

What Remix Plus adds or changes compared to upstream `dxvk-remix`. [`docs/CHANGELOG.md`](docs/CHANGELOG.md) has
the details.

- **A generic backend.** The renderer sits behind an API-neutral scene interface with a retained scene. The D3D9
  frontend and Remix API hosts both feed it.
- **A rebuilt D3D9 frontend.** The old D3D9 path is removed.
- **Remix API 1.0.0.** Retained instances, D3D11 host support, an external frame, color grading, host settings in
  the Remix menu, GPU texture import, screen overlays and more. Plugins built against the `rebased-9-19` API keep
  working.
- **The NUMOS sky.** A physically based atmosphere, volumetric clouds, a night sky and weather (`rtx.skyMode = 1`),
  from the separate NUMOS project.
- **Faster defaults.** SHaRC indirect lighting (`rtx.integrateIndirectMode = 3`) and light BVH light sampling
  (`rtx.lightSampling.mode = 1`) are on by default, and static geometry is merged into stable BLAS clusters.
- **Fewer stalls.** A persistent pipeline cache (`rtx.pipelineCache.persist`) and shader prewarming at startup.
- **A faster 32-bit bridge** for 32-bit D3D9 games.
- **Tone mapping and output.** A new tone curve, HDR10 output when Windows has HDR on, color grade presets
  (`rtx.tonemap.gradePreset`, preset files in `rtx-remix/colorgrade`), and screenshots with Print Screen.
- **The Alt+X menu.** A reworked user menu for players, with the developer menu behind it.
- **Diagnostics.** Detailed performance logging (`rtx.profile.detailedLogging`), a crash notice that points at the
  logs and crash dumps, and a hardened release build option (`-Drelease_build=true`, the default).

Every option is listed in [`RtxOptions.md`](RtxOptions.md).

## Supported games and hosts

- **32-bit D3D9 games**, through the bridge: a 32-bit `d3d9.dll` in the game folder starts a 64-bit server that
  runs the runtime from the `.trex` folder.
- **64-bit D3D9 games**, with the runtime's `d3d9.dll` next to the game's executable.
- **Remix API hosts**: games, emulators and plugins that call the Remix C API
  ([`public/include/remix/remix_c.h`](public/include/remix/remix_c.h)). A host can render through the API's own
  D3D9 device, from its own D3D11 device, or hand over an external frame.

Only x64 Windows is built and tested. Remix needs a GPU and driver with Vulkan ray tracing support.

## Install (players)

1. Download the latest Remix Plus zip from this repository's
   [Releases](https://github.com/RemixProjGroup/dxvk-remix/releases) page.
2. Extract it into the game's folder, next to the game's executable, and let it replace existing files.
   - For a 32-bit game, the zip has the bridge's `d3d9.dll` and launcher at the top and the runtime in `.trex`.
   - For a 64-bit game, the runtime files go next to the executable.
3. Start the game. Press **Alt+X** to open the Remix menu.

Copy every file in the zip. The runtime loads dozens of DLLs that ship beside `d3d9.dll`, and a partial copy fails to
load. Remix mods go in `rtx-remix/mods/`; see
[`documentation/RemixModsForPlayers.md`](documentation/RemixModsForPlayers.md).

## Build from source

Requirements:

- Windows 10 or 11.
- Visual Studio 2026 (VS 18) or its Build Tools, with the x64 C++ tools. `scripts/build.ps1` prefers VS 18 and falls
  back to VS 2022 or 2019. The bridge builds with VS 2019 or 2022.
- [Windows SDK 10.0.19041.0](https://developer.microsoft.com/en-us/windows/downloads/sdk-archive/) or newer.
- [Meson 1.8.2+](https://mesonbuild.com/) and Ninja (`pip install --user meson ninja`).
- [Vulkan SDK 1.4.313.2+](https://vulkan.lunarg.com/sdk/home#windows).
- [Python 3.9+](https://www.python.org/downloads/) (not the Microsoft Store version).
- Internet access for the first configure: it downloads the prebuilt dependencies listed in `packman-external.xml`
  into `external/`.

Get the code with its submodules:

```powershell
git clone --recursive https://github.com/RemixProjGroup/dxvk-remix.git
cd dxvk-remix
git submodule update --init --recursive   # if you cloned without --recursive
```

Build the runtime (64-bit `d3d9.dll`):

```powershell
.\scripts\build.ps1                                   # release build into _Comp64Release, installed to _output
.\scripts\build.ps1 -Flavor debugoptimized            # or debug
.\scripts\build.ps1 -Clean                            # fresh build dir
.\scripts\build.ps1 -MesonArgs '-Denable_d3d11=true','-Denable_dxgi=true'
```

- `-Denable_d3d11=true -Denable_dxgi=true` builds DXVK's D3D11 and DXGI modules, which Remix API hosts that render
  with D3D11 need. Both are off by default.
- `-Drelease_build=true` (the default) leaves local paths out of the binary, installs no PDB, EXP or LIB, and runs an
  extra SPIR-V optimization pass. Set it false for a development build.
- `-Dbuild_tag=<name>` prints `Build: <name>` once at startup in `remix-dxvk.log` (default `dev`).
- The script caps the job count at 4 (`-Jobs`), because more parallel jobs can run the compiler out of memory.

Build the bridge (32-bit client and 64-bit server) into `bridge/_output/`:

```powershell
.\bridge\scripts\build.ps1
```

To deploy builds to games automatically, copy `gametargets.example.conf` to `gametargets.conf` and edit the paths.
[`docs/CONTRIBUTING.md`](docs/CONTRIBUTING.md) has the full walkthrough.

## Documentation

- [Remix API reference](docs/RemixApi.md), with the [SDK setup](documentation/RemixSDK.md) and the
  [generated API surface](RemixApiSurface.md).
- [Migrating to the Remix Plus API](docs/RemixApi-Migration.md) from the `rebased-9-19` API, and the
  [host path performance guide](docs/api-host-path-migration.md).
- [Remix API changelog](documentation/RemixApiChangelog.md) and the [Remix Plus changelog](docs/CHANGELOG.md).
- [RtxOptions.md](RtxOptions.md): every runtime option, generated from the code.
- [Remix Sky API](docs/RemixSkyAPI.md), [RemixFX post-processing effects](docs/RemixFx.md) and the
  [post-processing stack](docs/PostProcessingStack.md).
- How features work, in [`documentation/`](documentation/): [Remix Config](documentation/RemixConfig.md),
  [Remix Logic](documentation/RemixLogic.md), [mod order](documentation/RemixModOrder.md),
  [mod packaging](documentation/RemixModPackaging.md), [SHaRC](documentation/SharcRadianceCache.md),
  [anti-culling](documentation/AntiCullingSystem.md), [terrain](documentation/TerrainSystem.md) and more.
- For contributors: [AGENTS.md](AGENTS.md) (the rules), [CONTRIBUTING](docs/CONTRIBUTING.md),
  [branch names](docs/BRANCHING.md), the [style guide](documentation/CONTRIBUTING-style-guide.md) and the
  [shader guide](src/dxvk/shaders/rtx/README.md).

## Reporting issues

Open an issue in this repository. Include:

- the game, how Remix is installed (bridge or 64-bit, any mods), your GPU and driver version;
- what you did, what you saw, and a screenshot if it is a rendering problem;
- the logs from the game's `rtx-remix/logs/` folder: `remix-dxvk.log`, and for a 32-bit game also `bridge32.log`
  and `bridge64.log`. After a relaunch, the previous session's logs are the `*.prev.log` files;
- after a crash, the newest `.dmp` file from `rtx-remix/logs/.sentry-native/reports/`. The crash notice that
  Remix shows on the next start opens that folder. Crash reports are never uploaded;
- your `rtx.conf` and `user.conf` if you changed settings.

## Credits and licenses

Remix Plus stands on the work of:

- [DXVK](https://github.com/doitsujin/dxvk) by Philip Rebohle and Joshua Ashton (zlib, [`LICENSE`](LICENSE)).
- [NVIDIA RTX Remix `dxvk-remix`](https://github.com/NVIDIAGameWorks/dxvk-remix) (MIT,
  [`LICENSE-MIT`](LICENSE-MIT)).
- [NUMOS](https://github.com/Kim2091/numos-sky), the sky, atmosphere and cloud renderer, a separate project by
  Kim2091 under the Apache License 2.0, used as the `submodules/numos` submodule.
- The tone curve by TheGreatHmmmmm (MIT), inspired by Prism by Jon for RenoDX (MIT).
- Code ported from other community forks, credited file by file in [`CREDITS.md`](CREDITS.md).

[`CREDITS.md`](CREDITS.md) says which license covers which code. Bundled libraries and SDKs are listed with their
licenses in [`ThirdPartyLicenses.txt`](ThirdPartyLicenses.txt) and
[`bridge/ThirdPartyLicenses.txt`](bridge/ThirdPartyLicenses.txt). The NVIDIA SDKs used as submodules (RTXDI, SHaRC,
NRC, RTXCR, DLSS) are used unmodified under their own licenses.

## Team

- [Kim2091](https://github.com/Kim2091): project lead and lead maintainer
- [CR](https://github.com/sambow23): maintainer
- [TheGreatHMMMM](https://github.com/TheGreatHMMMM): contributor
- [Gokuwashere](https://github.com/BrunchyChineapple): contributor

Questions? Open an issue or ask on the [RTX Remix Discord](https://discord.gg/c7J6gUhXMk).
