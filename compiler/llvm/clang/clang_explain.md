# **SETUP**

## **clang\_main**

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

## **BuildCompilation**

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

## **Compilation** class

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

## **BuildJobs**

> **Action**: An abstract compilation step (a node in the DAG).

**Example:**

```bash
clang foo.c bar.c
```

* **InputAction(foo.c)**: encapsulates the source file `foo.c`
* **InputAction(bar.c)**: encapsulates the source file `bar.c`
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

## **ExecuteCompilation**

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

##  **ExecuteJobs**

In Unix, all jobs are executed regardless of whether an error occurs
In MSVC’s `CLMode`, jobs stop when an error occurs

on all the jobs :
 * call the function ExecuteCommand
 * if fail store in the failing command vector
    * if CLMode return 

##   **ExecuteCommand**

* print if `CC_PRINT_OPTIONS`
* Execute `C.Execute`
* manage error

##   **Execute**

* **Print file names** for logging and diagnostics.
* **Construct `Argv`**:

  * If no response file: push `Executable`, optional `PrependArg`, then `Arguments`, terminate with `nullptr`.
  * If a response file is needed:

    1. **Serialize** arguments into `RespContents` via `writeResponseFile`.
    2. **Build** `Argv` for the response file (`@file` syntax).
    3. **Write** the response file with proper encoding; on error, set `ErrMsg`/`ExecutionFailed` and return `-1`.
* **Prepare environment** (`Env`) if any variables are set.
* **Convert** `Argv` array to `StringRef` array (`Args`).
* **Handle redirects**:

  * If `RedirectFiles` are present, convert to `std::optional<StringRef>` list and call `ExecuteAndWait` with those.
  * Otherwise, call `ExecuteAndWait` with the provided `Redirects`.
* **Execute and wait**:

  * `llvm::sys::ExecuteAndWait` forks, execs `Executable` with `Args` and `Env`, applies redirects, collects exit code in `ErrMsg`/`ExecutionFailed`, and records `ProcStat`.
* **Return** the child process exit code (or `-1` on exec failure).

## **Going back to the clang_start**

ok so lets resume we :
 * created the driver settings
 * with that we created the Compilation Settings
 * created actions for each part of the compilation
 * created a list of jobs from the list of actions

now it's time to execute the jobs !!!

in the begining we talk about a if the first args of the function is -cc1
that it we are building guys !!!

##   **ExecuteCC1Tool**

* tokenize the cmd line 
* redirect on the right cc1

## **cc1_main**

* init a CompilerInstance() and a DiagnosticIDs() instance

> * CompilerInstance
> this is the class responsible for handeling the `cc1`  part of clang
> it containt all the class needed by the compilation

> * DiagnosticIDs
> talk for imself that the class responsible for writing compilation diagnostic

* init PCH format that are precompile header
* init all taget base function
* dig setup
* fill the `CompilerInvocation` of the `CompilerInstance` that contain all compiler settings and little check on args list
* timeTraceProfile init if need
* print of clang cpu/stats
* init of the dig for the `CompilerInstance`
* install in the instance the llvm backend diag
* execute `ExecuteCompilerInvocation`
* handle error and timer

## **ExecuteCompilerInvocation**

* handle basic `-help` / `-version`
* load clang user plugin
* handle `-mllvm`
* optional satic analyser stuf
* error checkup
* creation of the `FontendAction` (bind the right ExecuteAction function on the FrontedAction)
* execute the `FontendAction`

## **ExecuteAction**

* **Preconditions**: Verify diagnostics are initialized and help/version have been handled.
* **Diagnostics cleanup guard**: Ensure `getDiagnosticClient().finish()` runs on exit.
* **Verbose stream**: Obtain the verbose output stream (`OS`).
* **Prepare action**: Invoke `Act.PrepareToExecute(*this)`, which sets up any internal state required by the action; if it returns `false`, the action cannot run and `ExecuteAction` immediately returns `false` to signal failure (rather than continuing into an invalid state).
* **Create target**: Initialize the compilation target (the architecture, vendor, OS, and ABI settings—e.g., `x86_64-unknown-linux-gnu`), configuring TargetInfo/TargetMachine; abort on failure.
* **ObjC rewrite patch**: For Objective-C (`ObjC`) rewriting actions (e.g., `RewriteObjC`), adjust the built-in ObjCBool type (disable signed char) to match the ObjC ABI.
* **Verbose/stats flags**: Print version info if verbose, enable stats if requested.
* **Sort codegen tables**: Sort TOC (`Table of Contents`) and NoTOC variable lists (used on targets like PowerPC64 for PIC data) so lookups use binary search for efficient codegen decisions.
* **Process inputs**: For each input file, clear IDs, run `BeginSourceFile`, `Execute`, and `EndSourceFile`.
* **Print diagnostic stats**: Emit any collected diagnostic statistics.
* **Dump stats to file**: If `StatsFile` set, open (or append) and write JSON stats, warning on error.
* **Return success**: Return true if no errors were reported, false otherwise.

## **Execute**

* **Get CompilerInstance**
* **ExecuteAction**: Invoke the action-specific frontend logic (`ExecuteAction()`).
* **Rebuild Global Module Index**: If `CI.shouldBuildGlobalModuleIndex()` and file manager/preprocessor are present, fetch `Cache = CI.getPreprocessor().getHeaderSearchInfo().getModuleCachePath()` and, if non-empty, call `GlobalModuleIndex::writeIndex`. On error, consume it silently.
* **Return success**: Always return `llvm::Error::success()` (no error propagation).

# **ASTcreation**

## **FrontendAction Hierarchy**

Clang’s frontend actions all inherit from the abstract base `FrontendAction`, providing a uniform `ExecuteAction` entry point. Key subclasses include:

* **ASTFrontendAction**: Runs after parsing to operate on the AST (analysis, transformations).
* **PreprocessorFrontendAction**: Hooks into the preprocessor stage (token processing before parsing).
* **CodeGenAction**: Coordinates code generation backends (e.g., IR emission, object code output).
* **Other FrontendActions**: Miscellaneous actions (e.g., `MergeModuleAction`, `PluginAction`).

Below is the `EmitLLVMAction`, which inherits from `CodeGenAction` and emits LLVM IR:

```cpp
class EmitLLVMAction : public CodeGenAction {
  virtual void anchor();              // Ensure vtable emission
public:
  EmitLLVMAction(llvm::LLVMContext *_VMContext = nullptr);
};

void EmitLLVMAction::anchor() { }
EmitLLVMAction::EmitLLVMAction(llvm::LLVMContext *_VMContext)
  : CodeGenAction(Backend_EmitLL, _VMContext) {}
```

**What this does**:

1. **anchor()**: Defines an out‑of‑line virtual method so that the compiler emits `EmitLLVMAction`’s vtable in this translation unit.
2. **Constructor**: Calls the `CodeGenAction` base with `Backend_EmitLL`, registering the action to generate LLVM IR into the given (or newly created) `LLVMContext`.

> note : all the CodeGenAction are all the same with the `llvm::LLVMContext` (`Backend_EmitLL`) that change

