# **clang\_main**

* **Setup bug report message**
* On Windows, reopen any missing `stdin`/`stdout`/`stderr` handles to `NUL`
* **Initialize all architecture targets** (x86, ARM, AArch64, etc.)
* **Create** a `BumpPtrAllocator` and `StringSaver` to own strings from `@file` response files
* **Get** the program name (e.g. `bin/clang`)
* **Determine** if arguments are MSVC-style (`clang-cl` mode)
* **`expandResponseFiles`**: splice tokens from any `@file` entries into the argument list
* **If** this is a `-cc1` invocation (handled later via `ExecuteCC1Tool`)
* **Parse early settings** like `-canonical-prefixes` / `-no-canonical-prefixes`
* **Handle** CL-mode environment variables (`CL`, `_CL_`) for MSVC overrides
* **Apply** `CCC_OVERRIDE_OPTIONS` overrides (e.g. `CCC_OVERRIDE_OPTIONS="# O0 +-g"`)

  * `#` silences the “### Removing …” / “### Adding …” messages
  * Replaces any `-O*` flags with `-O0`
  * Appends `-g` to the flag list
* **GetExecutablePath**: compute the absolute or raw driver path
* **Parse** `-fintegrated-cc1` / `-fno-integrated-cc1` to choose in-process vs. external cc1
* **Setup** the diagnostic engine

  * **Parse** DiagOpts (`-Werror`, `-pedantic`, etc.)
  * **Instantiate** `TextDiagnosticPrinter` and `DiagnosticsEngine`
* **Initialize** the filesystem VFS and process warning options
* **Initialize** `TheDriver` (GCC-like driver main body)
* **Configure** extra arguments (target, mode, prefix)
* **Assign** `TheDriver.CC1Main = ExecuteCC1WithContext` if using in-process cc1
* **Build** the `Compilation` via `BuildCompilation` (preprocess, compile, assemble, link)
* **Handle** errors/crashes during `Compilation` creation (Windows specifics, reproducers)
* **Execute** compilation jobs: `TheDriver.ExecuteCompilation(*C, FailingCommands)`
* **Handle** compilation errors/crashes and emit diagnostics

---

# **BuildCompilation**

* **Get driver mode** (`--driver-mode=g++`)

  Default mapping:

  ```bash
  clang foo.c      # DriverMode == "gcc"
  clang++ foo.cpp  # DriverMode == "g++"
  clang -E foo.c   # DriverMode == "cpp"
  clang-cl foo.c   # DriverMode == "cl"
  ```

* **Parse** command-line arguments into `CLOptions`

* **Load** config file (`~/.config/clang/config`) via `loadConfigFiles`

  **Order of override**:

  1. Head defaults (`CfgOptionsHead`)
  2. User arguments (`CLOptions`)
  3. Tail overrides (`CfgOptionsTail`)

* **Fuse** head + user + tail into a single argument set

* **Translate** input args via `TranslateInputArgs(*UArgs)` → `TranslatedArgs`

* **Claim** flags to suppress unused warnings:

  * `-canonical-prefixes` / `-no-canonical-prefixes`
  * `-fintegrated-cc1` / `-fno-integrated-cc1`

* **Handle** hidden debug flags (`-ccc-print-phases`, `-ccc-print-bindings`)

* **Setup** MSVC or DXC/HLSL→Vulkan/SPIR-V modes

* **Configure** target and install directories (`COMPILER_PATH`, etc.)

* **Compute** `TargetTriple` from `--target`, `-m*`, driver-mode

* **Select** toolchain via `getToolChain`

* **Validate/warn** on triple vs. object-format mismatches

* **Append** multilib macros from `TC.getMultilibMacroDefinesStr`

* **Invoke** phases: preprocess → compile → assemble → link

* **Apply** architecture-specific settings (\~50 lines)

* **Initialize** the `Compilation` class

* **Call** `BuildJobs(*C)` to schedule compile processes

---

# **Compilation** class

```cpp
Compilation::Compilation(const Driver &D,
                         const ToolChain &_DefaultToolchain,
                         InputArgList *_Args,
                         DerivedArgList *_TranslatedArgs,
                         bool ContainsError)
    : TheDriver(D),
      DefaultToolchain(_DefaultToolchain),
      Args(_Args),
      TranslatedArgs(_TranslatedArgs),
      ContainsError(ContainsError) {
  // The offloading host toolchain is the default toolchain.
  OrderedOffloadingToolchains.insert(
      std::make_pair(Action::OFK_Host, &DefaultToolchain));
}
```

* **TheDriver**: Reference to the `Driver` controlling this compilation.
* **DefaultToolchain**: Host `ToolChain` for codegen.
* **Args** / **TranslatedArgs**: Raw and translated argument lists.
* **ContainsError**: Error flag from initial parsing.
* **OrderedOffloadingToolchains**: Inserts **`Action::OFK_Host`** → host toolchain.

---

# **BuildJobs**

> **Action**: An abstract compilation step (a node in the DAG).

**Example:**

```bash
clang foo.c bar.c
```

* **InputAction(foo.c)**: wrap `foo.c`
* **InputAction(bar.c)**: wrap `bar.c`
* **CompileJobAction(foo.c → foo.o)**: compile `foo.c`
* **CompileJobAction(bar.c → bar.o)**: compile `bar.c`
* **LinkJobAction(foo.o, bar.o → a.out)**: link objects

**Driver::BuildJobs steps:**

1. **Count outputs** (including `.ifs/.ifso` exceptions for `-o`)
2. **Handle** Mach-O multi-arch (`-arch`) specifics
3. **For each Action**:

   * Determine `LinkingOutput` for `LipoJobAction`
   * Call `BuildJobsForAction` to create Jobs
4. **Disable integrated-cc1** if `C.getJobs().size() > 1`
5. **Print stats** if `CCPrintProcessStats` is enabled
6. **Return** based on errors or `-Qunused-arguments`
7. **Suppress** warnings for flags like `-fdriver-only`, `-###`, etc.
8. **Warn** on any remaining unclaimed or unsupported arguments

## ExecuteCompilation

* **LogOnly mode**: if `-fdriver-only`, print jobs (`-v`) then execute in *log-only* mode.
* **Dry run**: if `-###`, print jobs and exit based on diagnostic errors.
* **Error check**: abort early if any diagnostic errors before execution.
* **Response files**: set up response files for each job when needed.
* **Execute jobs**: run `C.ExecuteJobs`, collecting failures in `FailingCommands`.
* **Fast exit**: return `0` when there are no failures.
* **Cleanup**: on failure and when not saving temps, remove produced result files; for crashes (`<0`), also clean failure files.
* **Signal handling**: ignore `EX_IOERR` (SIGPIPE) without printing diagnostics.
* **Detailed diagnostics**: if a failing tool lacks good diagnostics or exit code ≠1, emit `err_drv_command_failed` or `err_drv_command_signalled`.
* **Return code**: propagate the first non-zero or special exit code in `Res`.
