# **clang Frontend explain**

i welcome you to this walkthrough of the clang frontend execution!!! this walkthrough is mainly for me to dive into the clang source code, but if it helps others along the way, that’s a win!!!

**note:** to get the most out of this guide, follow along with the source code open at your side and spot what’s happening in real time.
**note:** i would not go into detail for `logging`, `error handeling`, `windows suppoort` and `objectif c specific` and this is manly a focus on c to remove some c++ overhead

i've added links to the github source code in each function explanation title for convenience.

# **Driver-Level Setup**

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

``` cpp
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

# **Compiler Core `cc1`**

## **Going back to the clang_start**

ok so lets resume we :
 * created the driver settings
 * with that we created the Compilation Settings
 * created actions for each part of the compilation
 * created a list of jobs from the list of actions

now it's time to execute the jobs !!!

in the begining we talk about a `if` the first args of the function is -cc1
that it we are building guys !!!

## **main class explain**
before we satrt if think is best that we explain all the main class in the compilation
`CompilerInstance` `FileManager`, `SourceManager`, `PreProcessor`, `Lexer`
this is more of a reference part to better undertand the rest

### **`CompilerInstance`**
This is the class responsible for handeling the `cc1`  part of clang
it containt all the class needed by the compilation

main class inside the `CompilerInstance`
* `FileManager`
* `SourceManager`
* `TargetInfo`
* `PreProcessor`
* `ASTContext` `ASTConsumer` `ASTReader`

function :
``` c
bool CompilerInstance::ExecuteAction(FrontendAction &Act);
```

### **`FileManager`**

The `FileManager` class find/read file on disk or `VFS` and caches metadata open memorybuffer.
This return a `FileEntryRef`
> VFS: is an llvm abstraction that makes all file sources (disk files, in-memory buffers, archives) look like a single, uniform filesystem to the compiler.

main members inside the `FileManager`

* `FileSystem`
* `FileSystemOptions`
* `DenseMap` of real files by ID
* `DenseMap` of real directories by ID
* `SmallVector` of virtual file entries
* `SmallVector` of virtual directory entries
* `StringMap` for cached file lookups
* `StringMap` for cached directory lookups
* `OptionalFileEntryRef` for stdin

function :

```cpp
llvm::Expected<FileEntryRef> FileManager::getFileRef(StringRef Filename,
                                                    bool OpenFile = false,
                                                    bool CacheFailure = true,
                                                    bool IsText = true);
```

### **`SourceManager`**

This class takes raw buffers from FileManager (`FileEntryRef`), assigns them simple integer FileIDs, and builds the data structures needed for the rest of the compiler to ask, “What line/column is offset N in FileID X?”

main members inside the `SourceManager`

* `SourceMgr` contains a list of `MemoryBuffer` objects (one per `FileID`)
* `FileIDMap` mapping from buffer index to `FileEntryRef`
* Line/column mapping tables for each buffer (e.g., `LineOffsets`)
* Macro expansion and include-location stacks
* `FileID` counter to assign unique IDs for each buffer

function :

```cpp
/// Create a new FileID for the given FileEntryRef and return it.
FileID SourceManager::createFileID(FileEntryRef FE,
                                   SourceLocation Loc,
                                   SrcMgr::CharacteristicKind K);

/// Retrieve character data for a given FileID and offset.
const char *SourceManager::getCharacterData(FileID FID, unsigned &Offset);
```

### **`Preprocessor`**

This class handles all pre‐parsing work—macro definitions, `#include` directives, conditional compilation—and produces a stream of tokens for the parser.

main members inside the `Preprocessor`

* `HeaderSearch`            – manages search paths and maps headers to `FileEntryRef`
* `PreprocessorOptions`     – stores flags like `-D`, `-I`, macro expansion settings
* `IdentifierTable`         – uniquing table for identifiers and keywords
* `Builtin::Context`        – definitions for built–in macros and keywords
* `MacroInfoMap`            – maps macro names to their definitions (`MacroInfo`)

functions :

```cpp
/// Get the next token, expanding macros and handling directives.
Token Preprocessor::Lex(Token &Tok);

/// Push a new file onto the include stack, creating a fresh Lexer.
void Preprocessor::EnterSourceFile(FileID FID, bool IsMacroFile,
                                   llvm::MemoryBuffer *Buffer);
```

### **`Lexer`**

This class reads characters from a source buffer (via `SourceManager`) and produces `Token` objects for the parser.

main members inside the `Lexer`

* `FileID`                         – identifies the buffer being lexed
* `SourceManager &SM`             – provides access to buffer contents and locations
* `LangOptions &LangOpts`          – controls language-specific lexing behavior
* `const char *BufferStart/Ptr/End` – pointers to track current position in the memory buffer
* `Token CurToken`                 – storage for the current token
* `unsigned CurPPEnd`              – offset for end-of-macro/file switching

functions :

```cpp
/// Lex the next token from the buffer (skips whitespace/comments).
Token Lexer::Lex(Token &Result);

/// Initialize the lexer for a new buffer.
void Lexer::Initialize(FileID FID,
                       const LangOptions &LangOpts,
                       SourceManager &SM,
                       bool IsAtStartOfFile);
```

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

# **init AST creation**

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

## **CodeGenAction::ExecuteAction**

* **Wrapper over AST path**: Delegates all non-LLVM-IR inputs straight to `ASTFrontendAction::ExecuteAction()`.
* **LLVM IR handling**: Intercepts LLVM IR inputs and runs the IR-specific emission pipeline (bypassing the AST frontend).

> We’re not detailing the LLVM IR setup and configuration steps, as those belong to the backend-specific flow.

## **ASTFrontendAction::ExecuteAction**

* **Preprocessor check**: Return early if the preprocessor isn’t initialized.
* **Stack setup**: Mark bottom of the stack to guard against deep AST recursion.
* **Code completion**: If requested, install a `CodeCompleteConsumer` for IDE-style suggestions.
* **Semantic analysis init**: Create/configure the `Sema` object to drive name lookup and type checking.
* **Parse AST**: Invoke `ParseAST`, which lexes, parses, and executes the specific frontend action logic on the AST.

# **Parsing Class**

### **`Parser`**
The `Parser` class is responsible for syntactic parsing. It takes references to the `PreProcessor` and `Sema`, and drives the parsing of tokens into declarations and statements.

### **`Sema`**
The `Sema` (semantic analyzer) performs semantic checks (e.g., type checking, declaration validation). It owns references to essential components used during semantic analysis:

main members inside the `Sema`
* `ASTContext` – manages all AST nodes and semantic info
* `PreProcessor` – token stream handler
* `LangOptions` – holds active language dialect flags
* `DiagnosticsEngine` – emits warnings and errors
* `SourceManager` – tracks source locations


### **`Scope`**
Tracks the current scope during parsing and semantic analysis. It helps `Sema` resolve names (variables, functions, types) correctly depending on the nesting level (e.g., inside functions, loops, conditionals, or namespaces).

For example:

```c
int foo() {
    while (1) {
        if (bar) {
            foobar();
        }
    }
}
```
`foobar` is in the `if` scope, which is in the `while` scope, which is in the function scope.

### **`Decl`**
`Decl` is the base class for all AST nodes representing C/C++ declarations (e.g., functions, variables, classes).
All declaration nodes (like `FunctionDecl`, `VarDecl`, `RecordDecl`) derive from `Decl`.

`DeclGroupRef` is a container used when multiple declarations are parsed together (e.g., `int a, b;`).

### **`ASTContext`**
`ASTContext` manages the lifetime and storage of all AST nodes and semantic information. It contains information about:

main members inside the `ASTContext`
* `Decl` nodes (all AST nodes)
* `LangOpts`
* `TargetInfo`

### **`ASTConsumer`**
A callback interface class to observe and process AST nodes as they’re built.

### **`Parser::ModuleImportState`**
An enum only for C++20 `module`/`import` that can only be placed at the start of a file. After the first declaration, the keywords `module`/`import` become normal identifiers that you can use.

example
``` cpp
module;
import foo; // valid here
int x;
import bar; // now invalid
int import = 1; // valid
```

# **parsing logic overview**

## **clang::ParseAST**

* Setup statistics system (used for diagnostics and performance tracking)
* Initialize `Sema`
* Create the `ASTConsumer` from `Sema`, which will receive the AST nodes
* Construct the `Parser`, connecting the `Preprocessor` and `Sema`
* Register crash recovery cleanup routines
* Ensure the lexer is available
* init the parsing logic, this is where the first token is read and first scop set (`DeclScope`)
* parsing logic int the HandleTopLevelDecl that in your case emit llvm ir
* process things like pragma weak
* finialize (need more desc)
* print stats


### **parsing logic**
this is the main parsing logic of the clang `Parser`.

``` cpp
    Parser::DeclGroupPtrTy ADecl; // tmp for the curent Decl
    Sema::ModuleImportState ImportState; // import state for c++20

    for (bool AtEOF = P.ParseFirstTopLevelDecl(ADecl, ImportState); !AtEOF;
         AtEOF = P.ParseTopLevelDecl(ADecl, ImportState)) {

      if (ADecl && !Consumer->HandleTopLevelDecl(ADecl.get()))
        return;
    }
```
`ParseFirstTopLevelDecl` is a wrapper arround `ParseTopLevelDecl` that init the `ImportState` for c++20
so this boucle return a Decl and while this is not AtEOF the Decl goes to the `Consumer->HandleTopLevelDecl` that is the `ASTConsumer` that transform in to the taget in youre case ir

## **Parser::ParseTopLevelDecl**
* setup of a destructor for the Parser
* giant switch case for special case (is there that `tok::eof` return true)
* init the `ParsedAttributes` for gnu style attr (`__attribute__((foo))`) and c++11 style (`[[foo]]`)
* parse the trailing attribute of both types
* `ParseExeternalDeclaration`
* handeling of ImportState for c++20

## **Parser::ParseExeternalDeclaration**
this function is to now if you need to redirect to `ParseDeclarationOrFunctionDefinition` or `ParseDeclaration` and handle special case like `asm` or `import`/`export`
it could be simpilfy like this :

```cpp
    if (Tok.isEditorPlaceholder()) {
      ConsumeToken();
      return nullptr;
    }
    if (getLangOpts().IncrementalExtensions &&
        !isDeclarationStatement(/*DisambiguatingWithExpression=*/true))
      return ParseTopLevelStmtDecl();

    // We can't tell whether this is a function-definition or declaration yet.
    if (!SingleDecl)
      return ParseDeclarationOrFunctionDefinition(Attrs, DeclSpecAttrs, DS);
  }
  // This routine returns a DeclGroup, if the thing we parsed only contains a
  // single decl, convert it now.
  return Actions.ConvertDeclToDeclGroup(SingleDecl);

```

the `ParseTopLevelStmtDecl` is for `clang-repl` so you can use c like you would with python command line
``` c
 ➜  ~ clang-repl
clang-repl> #include <stdio.h>
clang-repl> int foo = 5;
clang-repl> printf("foo = %d\n", foo);
foo = 5
clang-repl> foo = 2;
clang-repl> printf("foo = %d\n", foo);
foo = 2
```

* `ParseDeclaration` Parses a declaration only (variables, typedefs, namespaces, inline namespaces, etc.). this is use in the big switch case
* `ParseDeclarationOrFunctionDefinition` declaration or function body, depending on what follows the declarator.

## **ParseDeclarationOrFunctionDefinition**
the `ParsingDeclSpec (DS)` passed by `ParseExeternalDeclaration` by is set to nullptr so we are not using the `if (DS)` part.
but first you might say what is `PasingDeclSpec` ??

### **ParsingDeclSpec**

As stated in the class definition, `ParsingDeclSpec` is for  parsing a `DeclSpec`:

```cpp
/// A class for parsing a DeclSpec.
class ParsingDeclSpec : public DeclSpec {
```
The constructor of ParsingDeclSpec calls the parser’s getAttrFactory(), providing exactly what DeclSpec needs for initialization. The key advantage of ParsingDeclSpec is that it also creates a ParsingDeclRAIIObject to manage diagnostics—accumulating any warnings or errors during parsing and ensuring they’re either committed or discarded when the object goes out of scope.

### **DeclSpec**
okok but what is `DeclSpec` ?

`DeclSpec` captures all the information about declaration specifiers:

* **Storage specifiers (`SCS`)**: `typedef`, `extern`, `static`, `auto`, `register`, etc.
* **Thread storage specifiers (`TSCS`)**: `__thread`, `thread_local`, `_Thread_local`.
* **Type qualifiers (`TQ`)**: `const`, `volatile`, `restrict`, `atomic`, etc.
* **attribte**

Each of these categories is stored in a compact bitfield along with source-location metadata, allowing Clang to validate and diagnose specifier usage precisely.

ok now back to the `ParseDeclarationOrFunctionDefinition`
* we now init a `ObjCDeclContextSwitch ` that we don't realy care because this is for objectif-C context
* enter `ParseDeclOrFunctionDefInternal`


## **ParseDeclOrFunctionDefInternal**

* add the `DeclSpecAttrs` (`__attribute__`) list to the `ParsingDeclSpec`
* prepare for MS specific parsing
* parse freestanding declaration specifier  `typedef` `extern` `static` `auto` `register`, `class {}`, `struct {}` `enum {}` and fill the **DeclSpec**
* check if missing a `;` and parse trailing arg like `attribute`
* switch case there is a `;`
    * the swich case return the size of the keyword for diagnostic in `LengthOfTSTToken`
    * Suggest correct location to fix '[[attrib]] struct' to 'struct [[attrib]]'
    * change `Attrs` (`[[attribute]]`) position if possible
    * consume the `;`
    * create the AST (`Sema::ParsedFreeStandingDeclSpec`)
    * if `RecordDecl` (`struct`/`union` and `class` because `CXXRecordDecl` is a subclasse of `RecordDecl`) verify if valid
    * create a `DeclGroupPtrTy`
* if a struct has been parsed but not freestanding (`struct S { … } x;`) notify sema "hey, I just defined struct S,” before trying to parse x."
* attributes before an Objective-C `@interface`/`@protocol`/`@implementation`
* DS.abort (free RAII ParsingDeclSpec  ?)
* add `Attrs` (`[[attribute]]`) to the `DeclSpec`
* again object-c specific
* Detect `extern "c"`
* change `[[attribute]]` position if possible
* call `ParseDeclGroup`







function that need to be detail
- `ParseDeclarationSpecifiers`
- `DiagnoseMissingSemiAfterTagDefinition`
- `ParsedFreeStandingDeclSpec`
- `ActOnDefinedDeclarationSpecifier`


