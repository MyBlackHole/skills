---
name: xmake-build-system
description: Use when writing build configuration for C/C++ projects, or when the user mentions xmake as the build system. Use for creating, modifying, or debugging xmake.lua files. Use when the project needs package/dependency management with add_requires/add_packages.
---

# Xmake Build System

## Overview

Xmake is a Lua-based cross-platform build system with built-in package management. The build configuration is written in `xmake.lua` (not CMakeLists.txt, not Makefile). Unlike CMake's custom DSL, xmake uses native Lua syntax.

**Core principle:** `xmake.lua` is a Lua script, not a declarative config file. Every `target()` call defines a build target.

## When to Use

- User says "use xmake" or mentions `xmake.lua`
- Setting up C/C++ project build system
- Need package/dependency management (`add_requires` + `add_packages`)
- Cross-platform C/C++ projects (Linux, macOS, Windows, Android, iOS)
- Projects that would otherwise use CMake, Meson, or Makefile

**DO NOT use for:**
- Non-C/C++ projects (xmake focuses on compiled languages)
- Projects where the user explicitly prefers CMake or another build system
- Simple single-file compilation (just use compiler directly)

## Quick Reference

### xmake.lua Structure

```lua
-- Global settings
set_project("myapp")
set_version("1.0.0")
set_languages("cxx17")
add_rules("mode.debug", "mode.release")

-- Declare dependencies (global scope)
add_requires("fmt >=10.0.0")
add_requires("nlohmann_json")

-- Define targets
target("myapp")
    set_kind("binary")
    add_files("src/*.cpp")
    add_includedirs("include")
    add_packages("fmt", "nlohmann_json")
    add_syslinks("pthread")

target("test_myapp")
    set_kind("binary")
    add_files("tests/*.cpp")
    add_packages("fmt")
```

### Common Commands

| Command | Short | Purpose |
|---------|-------|---------|
| `xmake` | `xmake b` | Build default target |
| `xmake build [target]` | `xmake b [t]` | Build specific target |
| `xmake config` | `xmake f` | Configure build options |
| `xmake run [target]` | `xmake r [t]` | Run a target |
| `xmake clean` | `xmake c` | Clean build artifacts |
| `xmake install` | `xmake i` | Install to system |
| `xmake create` | — | Create new project |
| `xmake -v` | — | Verbose build output |

### Configuration Options

```bash
xmake f -m debug          # Debug mode
xmake f -m release        # Release mode
xmake f -p linux          # Target platform
xmake f -a x86_64         # Target architecture
xmake f --toolchain=clang # Toolchain
```

## Core Patterns

### Pattern 1: Target Definition (NOT project())

xmake uses `target()`, NOT `project()`. There is no `project()` function in xmake.lua.

```lua
-- ✅ CORRECT: target() defines a build target
target("myapp")
    set_kind("binary")
    add_files("src/*.cpp")

-- ❌ WRONG: project() does NOT exist in xmake.lua
-- project("myapp")
--     add_files("src/*.cpp")
```

### Pattern 2: Package Management (add_requires + add_packages)

The two-step flow: `add_requires` (global) → `add_packages` (per-target).

```lua
-- Step 1: Declare dependencies globally
add_requires("fmt >=10.0.0")       -- from xmake repo
add_requires("nlohmann_json")      -- header-only library
add_requires("vcpkg::zlib")        -- from vcpkg
add_requires("brew::libomp")       -- from Homebrew

-- Step 2: Bind to specific targets
target("myapp")
    set_kind("binary")
    add_files("src/*.cpp")
    add_packages("fmt", "nlohmann_json")  -- ✅ add_packages, NOT add_links
    add_syslinks("curl", "pthread")        -- system libraries use add_syslinks
```

**Key rules:**
- `add_requires()` goes at **global scope** (before targets)
- `add_packages()` goes **inside each target** that needs the dependency
- System libraries (libcurl, pthread, dl, m) use `add_syslinks()`
- NEVER use `add_links()` for packages declared with `add_requires()` — that's for local/user libraries

### Pattern 3: Build Modes

```lua
-- ✅ CORRECT: Use built-in rules
add_rules("mode.debug", "mode.release")

-- This automatically defines:
--   xmake f -m debug   → debug symbols, no optimization
--   xmake f -m release → no debug symbols, full optimization

-- ❌ WRONG: Manual mode detection (brittle)
-- if is_mode("debug") then
--     set_symbols("debug")
--     set_optimize("none")
-- end
```

### Pattern 4: Common Target Kinds

```lua
target("myapp")
    set_kind("binary")       -- executable (main function required)

target("mylib_static")
    set_kind("static")       -- static library (.a / .lib)

target("mylib_shared")
    set_kind("shared")       -- shared library (.so / .dll / .dylib)

target("mylib_headeronly")
    set_kind("headeronly")   -- header-only (no compilation needed)
    add_headerfiles("include/**/*.h")  -- for installation

target("mylib_object")
    set_kind("object")       -- object file collection (no linking)
```

### Pattern 5: Target Dependencies

```lua
target("core")
    set_kind("static")
    add_files("core/*.cpp")

target("app")
    set_kind("binary")
    add_files("app/*.cpp")
    add_deps("core")           -- ✅ links core automatically
    -- add_deps ensures core builds first and links into app
```

## Complete Example

A complete xmake.lua for a realistic project with main program + tests:

```lua
-- xmake.lua
set_project("myapp")
set_version("0.1.0")
set_languages("cxx17")
add_rules("mode.debug", "mode.release")

-- Dependencies
add_requires("fmt >=10.0.0")
add_requires("nlohmann_json")

-- Main program
target("myapp")
    set_kind("binary")
    add_files("src/*.cpp")
    add_includedirs("include")
    add_packages("fmt", "nlohmann_json")
    add_syslinks("curl")

-- Tests
target("test_utils")
    set_kind("binary")
    add_files("tests/*.cpp", "src/utils.cpp")
    add_includedirs("include")
    add_packages("fmt")

-- Header-only dependency (just for install/export)
target("json_lib")
    set_kind("headeronly")
    add_headerfiles("lib/json/*.h")
```

## xmake.lua API Reference

### Global Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `set_project(name)` | Set project name | `set_project("myapp")` |
| `set_version(ver)` | Set project version | `set_version("1.0.0")` |
| `set_languages(std)` | Set language standard | `set_languages("cxx17", "c11")` |
| `add_rules(...)` | Add build rules | `add_rules("mode.debug", "mode.release")` |
| `add_requires(...)` | Declare dependencies | `add_requires("fmt >=10.0.0")` |

### Target Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `target(name)` | Define a target | `target("myapp")` |
| `set_kind(type)` | Set target type | `set_kind("binary")` |
| `add_files(glob)` | Add source files | `add_files("src/*.cpp")` |
| `add_includedirs(dir)` | Add include directories | `add_includedirs("include")` |
| `add_headerfiles(glob)` | Add header files (for headeronly) | `add_headerfiles("include/**")` |
| `add_packages(...)` | Bind declared packages | `add_packages("fmt")` |
| `add_syslinks(...)` | Add system libraries | `add_syslinks("pthread", "dl")` |
| `add_links(...)` | Add user/local libraries | `add_links("mylib")` |
| `add_linkdirs(dir)` | Add library search paths | `add_linkdirs("lib")` |
| `add_defines(...)` | Add preprocessor defines | `add_defines("DEBUG=1")` |
| `add_deps(...)` | Add target dependencies | `add_deps("core")` |

## Common Mistakes

### ❌ Using Wrong API Names (set_kind NOT kind, set_languages NOT language)

All xmake target configuration functions start with `set_` or `add_`. Some common mistakes:

```lua
-- ❌ WRONG: Missing set_ prefix
target("myapp")
    kind("binary")                    -- use set_kind("binary")
    language("c++17")                 -- use set_languages("cxx17")

-- ✅ CORRECT: Use proper set_ prefixed APIs
target("myapp")
    set_kind("binary")
    set_languages("cxx17")
```

**All target configuration APIs use `set_` (setters) or `add_` (adders) prefix:**
- `set_kind()` not `kind()`
- `set_languages()` not `language()`
- `add_files()` not `files()` or `source()`
- `add_deps()` not `add_dependencies()` or `depends_on()`
- `add_includedirs()` not `include_directories()` or `add_includes()`
- `set_platforms()` not `set_platform()` (note the 's')

### ❌ Placing Global APIs Inside target() Blocks

`add_requires()` and `add_rules()` must be at **global scope**, NOT inside `target()` blocks.

```lua
-- ❌ WRONG: add_requires and add_rules inside target()
target("myapp")
    add_rules("mode.debug", "mode.release")
    add_requires("fmt")
    set_kind("binary")
    add_files("src/*.cpp")
    add_packages("fmt")

-- ✅ CORRECT: Global APIs at top level, target APIs inside target()
add_rules("mode.debug", "mode.release")
add_requires("fmt")

target("myapp")
    set_kind("binary")
    add_files("src/*.cpp")
    add_packages("fmt")
```

**What goes where:**

| Global Scope (before any target) | Inside target() block |
|----------------------------------|----------------------|
| `add_rules(...)` | `set_kind(...)` |
| `add_requires(...)` | `add_files(...)` |
| `set_project(...)` | `add_includedirs(...)` |
| `set_version(...)` | `add_packages(...)` |
| `set_languages(...)` | `add_syslinks(...)` |
| | `add_links(...)` |
| | `add_defines(...)` |
| | `add_deps(...)` |

### ❌ Using `project()` Instead of `target()`

```lua
-- ❌ WRONG
project("myapp")
    set_kind("binary")
    add_files("src/*.cpp")

-- ✅ CORRECT
target("myapp")
    set_kind("binary")
    add_files("src/*.cpp")
```

**Why:** `project()` does not exist in xmake. xmake uses `target()` for each build artifact.

### ❌ Using `add_links()` Instead of `add_packages()`

```lua
add_requires("fmt")

target("myapp")
    add_files("src/*.cpp")
    -- ❌ WRONG: add_links is for local/user libraries, not remote packages
    add_links("fmt")

    -- ✅ CORRECT: add_packages binds the requires'd package
    add_packages("fmt")
```

**Why:** `add_requires` declares a remote dependency. `add_packages` binds it to a target. `add_links` is for local `.a/.so` files added with `add_linkdirs`.

### ❌ Placing Target APIs at Global Scope

```lua
-- ❌ WRONG: add_files and add_includedirs must be inside a target()
add_files("src/*.cpp")
add_includedirs("include")

target("myapp")
    set_kind("binary")

-- ✅ CORRECT
target("myapp")
    set_kind("binary")
    add_files("src/*.cpp")
    add_includedirs("include")
```

**Why:** `add_files`, `add_includedirs`, `add_packages` etc. must be inside a `target()` block. Only `add_requires`, `set_project`, `add_rules` are global.

### ❌ Forgetting `target()` Termination

While xmake doesn't require explicit `target_end()`, each `target()` call starts a new target definition. Make sure each target's settings are complete before the next `target()` call.

```lua
-- ✅ CORRECT: Each target's settings precede the next target() call
target("lib")
    set_kind("static")
    add_files("lib/*.cpp")

target("app")  -- ← lib target is implicitly terminated here
    set_kind("binary")
    add_files("app/*.cpp")
    add_deps("lib")
```

## Debugging xmake.lua

```bash
# Verbose build output
xmake -v

# Diagnostics
xmake f --diagnosis

# Test package detection
xmake l find_package fmt

# Show all configuration
xmake show

# Clean and reconfigure (fixes many issues)
xmake f -c && xmake f -m debug
```
