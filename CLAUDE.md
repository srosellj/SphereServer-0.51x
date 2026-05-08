# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# SphereServer 0.51x

C++ / Win32 fork of the recovered Sphere 0.51a UO server source (Menasoft, 1999-2000). The codebase is MFC-style legacy C++ targeting **Win32 x86 only** — no x64, no Linux build. T2A / pre-AOS era. Companion repo to `BlackoutUO` (a separate ModernUO/.NET 10 fork that ports these mechanics to C#) — **do NOT mix patterns between the two**.

The "wishlist" of in-flight and finished features lives in `README.md` (issues 01-30, with `[DONE]` markers).

## Solution layout (`GraySvr.sln`)

Single project `GraySvr/GraySvr.vcxproj`, two configurations: **`Debug|Win32`** and **`Release|Win32`**. The output binary is `GraySvr/Release/GraySvr.exe`.

- **`Common/`** — shared infrastructure compiled into both engine and tools: `cGrayMap` (map/multi tiles), `cFile` (file I/O), `cExpression` (script expression evaluator), `cRegion` (region tree), `cString` (in-house string class), `ccrypt` (client packet decryption), `cWindow` (Win32 console). Headers: `graycom.h` (precompiled, picks Win32 vs Linux paths but only Win32 is supported here), `graymul.h` (mul file format), `grayproto.h` (UO packet protocol), `graybase.h`.
- **`GraySvr/`** — the server itself:
  - World/objects: `CWorld`, `CSector`, `CChar*`, `CClient*`, `CItem*`, `CContain`, `CAccount`, `CChat`, `CParty`, `CMail`
  - Subsystems: `CCharFight` (combat), `CCharNPCAct`/`CCharNPCFood`/`CCharNPCPet` (NPC AI), `CClientMsg`/`CClientLog`/`CClientEvent`/`CClientTarg` (client interaction), `CItemSp` (special items), `CItemStone` (guild stones), `CVendorItem`, `CWorldImport`/`CWorldMap`/`CWorldFragment` (world serialization)
  - Win32 plumbing: `ntservice.cpp`, `ntwindow.cpp`, `GrayLog.cpp`
  - Entry point: `graysvr.cpp` / `graysvr.h` (precompiled header, defines `GRAY_TITLE`, `GRAY_VERSION="0.51a"`, `GRAY_DEF_PORT=2593`)
  - Resources: `GraySvr.rc`, `resource.h`, `spheredef.ini`
- **`OLDSTUFF/`** — historical bug reports / revisions. **Do not touch** unless explicitly asked.

`graysvr.h` requires `GRAY_SVR` to be defined on the compiler command line so the shared `Common/` headers compile correctly under both server and (legacy) client targets.

## Build

CI runs MSBuild on `windows-latest` and uploads `GraySvr/Release/GraySvr.exe` as an artifact (`.github/workflows/msbuild.yml`):

```cmd
nuget restore GraySvr.sln
msbuild /m /p:Configuration=Release GraySvr.sln
```

Local dev uses Visual Studio (originally VS2019, current `.sln` says Format 12 / VS16). Open `GraySvr.sln` and pick `Debug|Win32` or `Release|Win32`. Do not add x64 configurations.

There is no automated test suite. Validation is manual: launch `GraySvr.exe` against the script set (sphere `.scp` files) and connect with a 1999-era UO client.

## Editing rules

1. **Do NOT apply ModernUO/BlackoutUO rules here** — this is C++ legacy, not .NET. The LINQ tiers, `STArrayPool`, single-threaded event-loop, codegen serialization rules from `BlackoutUO/CLAUDE.md` are all irrelevant.
2. **Mantener estilo MFC clásico** — `C` prefix on classes, `m_` on members, raw pointers, `new`/`delete`. No introducir smart pointers, STL moderno ni features C++17/20 sin pedir confirmación.
3. **Win32 API directo** — `windows.h`, `winsock` (not winsock2), Win32 native threading. No portar a multiplataforma.
4. **Encoding** — los `.cpp`/`.h` originales pueden estar en CP-1252 / Windows-1252. Preservar el encoding original al editar.
5. **Sin tocar el protocolo de red sin aviso** — UO packet format está hardcoded; cualquier cambio rompe el cliente clásico. Confirmar antes.
6. **Build target**: `Debug|Win32` / `Release|Win32` exclusivamente.
7. **Era**: T2A / pre-AOS. No añadir features de expansiones posteriores.

## Relación con BlackoutUO

Aunque BlackoutUO (carpeta hermana en este checkout) también es un servidor UO, son **proyectos completamente independientes**:

- BlackoutUO: ModernUO fork, .NET 10, C#, single-threaded event loop, cross-platform.
- SphereServer-0.51x: Sphere fork, C++ MFC-style, Win32 x86, threading nativo.

This repo is sometimes used as a reference when tracing engine-side behavior of the Sphere 51a "official" model (e.g., where `m_skilllevel`-driven magic weapon bonuses are applied at hit time). Findings from such traces are written to `BlackoutUO/dev-docs/blackout/sphere51-combat-audit.md`, not here.
