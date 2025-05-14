# clang\_main

* setup bug report msg
* On Windows, reopen any missing `stdin`/`stdout`/`stderr` handles to `NUL`
* initialise all arch targets (x86, ARM, AArch64, etc.)
* create a `BumpPtrAllocator` and `StringSaver` to own strings read from `@file` (response files with compiler flags)
* get the program name (e.g. `bin/clang`)
* determine if args are MSVC-style (`clang-cl` mode)
* `expandResponseFiles` splice tokens from any `@file` entries into the argument list
* parse early settings like `-canonical-prefixes` / `-no-canonical-prefixes` (control resolving `.`/`..` in paths)
* handle CL-mode env vars (`CL` and `_CL_`) for MSVC-style overrides
* apply `CCC_OVERRIDE_OPTIONS` overrides (e.g. `CCC_OVERRIDE_OPTIONS="# O0 +-g"`) on the raw args list

  * `#` silences the “### Removing …” / “### Adding …” messages
  * replaces any `-O*` flags with `-O0`
  * appends `-g` to the flag list
* GetExecutablePath (compute the absolute or raw driver path)
* parse `-fintegrated-cc1` / `-fno-integrated-cc1` to decide between in-process and external cc1
* setup the diag engine

  * parse DiagOpts (`-Werror`, `-Wno-rewrite-macros`, `-pedantic`, etc.)
  * setup `TextDiagnosticPrinter`
  * instantiate `DiagnosticsEngine` with the options and printer
* init the filesystem VFS

  * Process warning options (grouping, suppressions)
* init TheDriver 😎 main body of a GCC-like driver for `cc1`
* set up any extra args (target, mode, prefix)
* set `TheDriver.CC1Main = ExecuteCC1WithContext` if using in-process cc1
* `BuildCompilation` main compilation process (preprocess, compile, assemble, link)
* remaining code handles errors/crashes (Windows specifics, reproducers)

# BuildCompilation

* **get driver mode** (`--driver-mode=g++`)

  default :

  ```
  clang foo.c      # DriverMode == "gcc"
  clang++ foo.cpp  # DriverMode == "g++"
  clang -E foo.c   # DriverMode == "cpp"
  clang-cl foo.c   # DriverMode == "cl"
  ```

* **parse cmd line arguments** into `CLOptions`

* **load config file** (.clang head & tail via `loadConfigFiles`) if no error in CLI parsing

**order of override**:

1. head (defaults) `CfgOptionsHead`
2. user args (can override) `CLOptions`
3. tail (forced overrides) `CfgOptionsTail`

```text
# ~/.config/clang/config

# Head defaults:
-std=c++20
-Wall

--        ← split marker

# Tail overrides:
-O2
```

* **fuse** head + user + tail
* **translate input args** via `TranslateInputArgs(*UArgs)` → `TranslatedArgs`
  (e.g., map `-pthread`, `-fopenmp`)
* **claim flags** (suppress unused warnings):

  * `-canonical-prefixes` / `-no-canonical-prefixes`
  * `-fintegrated-cc1` / `-fno-integrated-cc1`
* **handle hidden debug flags** (`-ccc-print-phases`, `-ccc-print-bindings`)
* **setup MSVC or DXC/HLSL→Vulkan/SPIR-V modes**
* **setup target and install dirs**, `COMPILER_PATH`, etc.
* **Compute `TargetTriple`** from `--target`, `-m*`, driver-mode tweaks
* **getToolChain** picks the appropriate `ToolChain`
* **validate/warn** on triple env vs object-format, arm64EC override, eabi/elf mix-ups
* **append multilib macros** from `TC.getMultilibMacroDefinesStr`
* **invoke phases** (preprocess → compile → assemble → link)
* **architecture-specific settings** (\~50 lines)

* **BuildJobs(\*C)**: schedule actual compile processes (the reall shiitt will start be patient)

# BuildJobs


