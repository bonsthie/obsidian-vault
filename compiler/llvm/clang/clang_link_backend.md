# **Backend Link**

We’ve parsed a complete top-level statement and now need to forward it to the backend for code generation. This section clarifies how Clang transitions from parsing to emitting IR via `CodeGenModule::EmitTopLevelDecl`:

### Parsing Loop (`clang::ParseAST`)

```cpp
for (bool AtEOF = P.ParseFirstTopLevelDecl(ADecl, ImportState);
     !AtEOF;
     AtEOF = P.ParseTopLevelDecl(ADecl, ImportState)) {
  if (ADecl && !Consumer->HandleTopLevelDecl(ADecl.get()))
    return;
}
```

* **`ParseFirstTopLevelDecl` / `ParseTopLevelDecl`**: Retrieve the next `DeclGroupRef` from the parser
* **`Consumer->HandleTopLevelDecl`**: Invokes the backend-specific handler with the parsed declaration

### Backend Dispatch Chain

1. **`BackendConsumer::HandleTopLevelDecl`**

   * Entry point from the frontend to the code-generation backend
2. **`CodeGeneratorImpl::HandleTopLevelDecl`**

   * Iterates over each `Decl` in the `DeclGroupRef`, passing them on to the builder
3. **`CodeGen::CodeGenModule::EmitTopLevelDecl`**

   * Final step to inspect and emit each individual `Decl`


### `CodeGenModule::EmitTopLevelDecl`

* **Dispatch Role**: Determines the nature of the `Decl` and delegates accordingly with a switch case
* **`EmitGlobal(GlobalDecl)`**: Converts `FunctionDecl`/`VarDecl` into LLVM IR objects
* **Contextual Processing**: For containers like namespaces or records, it may recurse


