# Agents Guide (`AGENTS.md`)

## Core Directive: JKA Multiplayer Scope Only

**IMPORTANT:** When the user refers to code, features, bugs, or changes, they are **exclusively referring to the Jedi Academy (JKA) Multiplayer (`codemp/`) portion of the codebase** unless explicitly stated otherwise.

### In-Scope Directories
- `codemp/` — Primary directory for all JKA Multiplayer code:
  - `codemp/game/` — Server-side game logic (`jampgame` module, compiled as C). Handles entities, NPCs, physics, combat, weapons, Force powers, client commands (`g_cmds.c`), and server commands (`g_svcmds.c`).
  - `codemp/cgame/` — Client-side game logic (`cgame` module, compiled as C). Handles client prediction, view rendering, local entities, HUD/effects, and server command parsing (`cg_servercmds.c`).
  - `codemp/ui/` — User interface and menus (`ui` module, compiled as C).
  - `codemp/client/` — MP Engine client subsystem (C++).
  - `codemp/server/` — MP Engine server subsystem (C++).
  - `codemp/qcommon/` — MP Engine common utilities, networking, Cvars, filesystems, and shared headers (`q_shared.h`).
  - `codemp/rd-vanilla/` & `codemp/rd-rend2/` & `codemp/rd-dedicated/` & `codemp/rd-common/` — MP Renderers.
  - `codemp/ghoul2/`, `codemp/icarus/`, `codemp/botlib/`, `codemp/Ratl/`, `codemp/Ravl/`, `codemp/mp3code/`, `codemp/null/`, `codemp/win32/`, `codemp/macosx/` — MP Engine subsystems and platform layers.
- `shared/` — Platform/math/string utilities shared across targets (modify with caution, keeping MP focus in mind).

### Out-of-Scope Directories (Do NOT Modify or Search Unless Explicitly Asked)
- `code/` — Jedi Academy Single Player (`openjk_sp`, `jagame`). **Ignore this directory.** Many files share names with `codemp/` (e.g., `g_main.cpp`, `NPC.cpp`, `cg_main.cpp`), so always verify your file paths start with `codemp/` and not `code/`.
- `codeJK2/` — Jedi Outcast Single Player (`openjo_sp`, `jospgame`). **Ignore this directory.**

---

## Architecture & Language Conventions (`codemp/`)

1. **C vs. C++ Split in `codemp/`**:
   - **Game Modules (`codemp/game/`, `codemp/cgame/`, `codemp/ui/`)**: Written in **C** (`.c` files). Do not introduce C++-only syntax, classes, templates, or standard library headers into `.c` files or headers included by `.c` files unless compiling under C++. Use Quake 3 / Raven C idioms (`qboolean`, `qtrue`, `qfalse`, `vec3_t`, `gentity_t`, `clientSession_t`, `trap_*` syscalls).
   - **Engine, Renderer, Botlib, Ghoul2, ICARUS (`codemp/client/`, `codemp/server/`, `codemp/qcommon/`, `codemp/rd-*/`, etc.)**: Written primarily in **C++** (`.cpp` files).

2. **Shared Game Code (`bg_*`)**:
   - Files prefixed with `bg_` (Both Games) in `codemp/game/` (e.g., `bg_misc.c`, `bg_pmove.c`, `bg_saber.c`, `bg_panimate.c`, `bg_public.h`, `bg_local.h`, `AnimalNPC.c`, `FighterNPC.c`, `SpeederNPC.c`, `WalkerNPC.c`) are compiled into **multiple targets** (`jampgame`, `cgame`, and partially `ui`).
   - Preprocessor guards dictate context:
     - `_GAME` — Defined when compiling `codemp/game` (`jampgame`).
     - `_CGAME` — Defined when compiling `codemp/cgame` (`cgame`).
     - `UI_BUILD` — Defined when compiling `codemp/ui` (`ui`).
   - When modifying `bg_*` files or headers, ensure compatibility across all modules that compile them.

3. **Engine-Module Communication**:
   - Game modules (`game`, `cgame`, `ui`) communicate with the engine via `trap->*` function pointers / `trap_*` syscalls (defined in `g_syscalls.c`, `cg_syscalls.c`, `ui_syscalls.c` and interfaced via `g_public.h`, `cg_public.h`, `ui_public.h`).

4. **Adding or Removing Source Files**:
   - File lists in CMake are **explicitly enumerated** (not globbed). If you add or remove `.c`, `.cpp`, or `.h` files, update the corresponding `CMakeLists.txt` (e.g., `codemp/CMakeLists.txt`, `codemp/game/CMakeLists.txt`, `codemp/cgame/CMakeLists.txt`, `codemp/ui/CMakeLists.txt`).

---

## Build System & Targets

The project uses CMake with a Visual Studio solution (`build/OpenJK.sln`, targeting `Win32` / `x86` by default on Windows).

### Relevant JKA MP CMake Targets
- `jampgamex86` (`BuildMPGame`) — Server-side game DLL (`codemp/game/`)
- `cgamex86` (`BuildMPCGame`) — Client-side game DLL (`codemp/cgame/`)
- `uix86` (`BuildMPUI`) — UI DLL (`codemp/ui/`)
- `openjk.x86` (`BuildMPEngine`) — MP client executable
- `openjkded.x86` (`BuildMPDed`) — MP dedicated server executable
- `rd-vanilla_x86` (`BuildMPRdVanilla`) — MP default renderer DLL
- `rd-rend2_x86` (`BuildMPRend2`) — MP experimental renderer DLL

### Building via Command Line
To build specific MP targets from the existing `build/` directory without building Single Player targets:
```powershell
cmake --build build --config Debug --target jampgamex86
cmake --build build --config Debug --target cgamex86
cmake --build build --config Debug --target uix86
cmake --build build --config Debug --target openjk.x86
```

---

## Code Style & Formatting

Follow `.editorconfig` and existing OpenJK conventions:
- **Indentation**: Tabs (`indent_style = tab`, `indent_size = 4`).
- **Line Endings / Whitespace**: Trim trailing whitespace and ensure a final newline.
- **String & Memory Safety**: Prefer safe string manipulation utilities (`Q_strncpyz`, `Com_sprintf`, `Q_stricmp`, `Q_strcat`) over raw standard C functions (`strcpy`, `sprintf`, `strcmp`).

