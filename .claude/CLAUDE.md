# SphereServer 0.51x

Sphere 0.51a UO server emulator (1999-2000 code recovered). C++ Win32 only.

- **Solution**: `GraySvr.sln` (Visual Studio 2019, **Win32 x86 únicamente** — no x64)
- **Common/**: infraestructura compartida (cGrayMap, cFile, cExpression, ccrypt)
- **GraySvr/**: lógica del servidor (CChar*, CClient*, CItem*, CContain, CChat)
- **OLDSTUFF/**: código histórico, no tocar salvo petición explícita

## Reglas de edición

1. **NO aplicar reglas de ModernUO/BlackoutUO aquí** — este es C++ legacy, no .NET. Las reglas del CLAUDE.md de BlackoutUO (LINQ tiers, STArrayPool, single-threaded, serialización codegen, etc.) son irrelevantes.
2. **Mantener estilo MFC clásico** — prefijo `C` en clases, `m_` en miembros, raw pointers, `new`/`delete`. No introducir smart pointers, STL moderno ni features C++17/20 sin pedir confirmación.
3. **Win32 API directo** — no portar a multiplataforma. Asumir `windows.h`, `winsock`, threading Win32 nativo.
4. **Encoding** — los `.cpp`/`.h` originales pueden estar en CP-1252 / Windows-1252. Preservar encoding original al editar.
5. **Sin tocar protocolo de red** — UO packet format hardcoded, cualquier cambio rompe el cliente clásico. Confirmar antes.
6. **Build target**: Debug|Win32 / Release|Win32. No añadir configuraciones x64.
7. **Era**: T2A / pre-AOS. No añadir features de expansiones posteriores.

## NO mezclar con BlackoutUO

Aunque BlackoutUO (carpeta hermana) también es un servidor UO, son **proyectos completamente independientes**:
- BlackoutUO: ModernUO fork, .NET 10, C#, single-threaded event loop, cross-platform
- SphereServer-0.51x: Sphere fork, C++ MFC-style, Win32 x86, threading nativo

No copiar patrones de uno al otro sin pedir confirmación explícita.
