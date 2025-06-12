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

* **`ParseFirstTopLevelDecl` / `ParseTopLevelDecl`**: Retrieve the next `DeclGroupRef` from the parser.
* **`Consumer->HandleTopLevelDecl`**: Invokes the backend-specific handler with the parsed declaration.

### Backend Dispatch Chain

1. **`BackendConsumer::HandleTopLevelDecl`**

   * Entry point from the frontend to the code-generation backend.
2. **`CodeGeneratorImpl::HandleTopLevelDecl`**

   * Iterates over each `Decl` in the `DeclGroupRef`, passing them on to the builder.
3. **`CodeGen::CodeGenModule::EmitTopLevelDecl`**

   * Final step to inspect and emit each individual `Decl`.


### `EmitTopLevelDecl` Dispatch Logic

This function uses a large `switch` on `Decl::Kind` to select how each declaration should be emitted:

```cpp
void CodeGenModule::EmitTopLevelDecl(Decl *D) {
  switch (D->getKind()) {
    case Decl::Function:
    case Decl::CXXMethod:
    case Decl::CXXConversion:
      EmitGlobal(cast<FunctionDecl>(D));
      AddDeferredUnusedCoverageMapping(D);
      break;

    case Decl::Var:
    case Decl::Decomposition:
    case Decl::VarTemplateSpecialization:
      EmitGlobal(cast<VarDecl>(D));
      break;

    case Decl::Namespace:
      EmitDeclContext(cast<NamespaceDecl>(D));
      break;

    case Decl::CXXRecord:
      // Emit class metadata, debug info, and recurse if needed
      break;

    // ... other Decl kinds ...

    default:
      // Decls like typedefs, templates, or using-directives are ignored
      break;
  }
}
```

* **Dispatch Role**: Determines the nature of the `Decl` and delegates accordingly.
* **`EmitGlobal(GlobalDecl)`**: Converts `FunctionDecl`/`VarDecl` into LLVM IR objects.
* **Contextual Processing**: For containers like namespaces or records, it may recurse.


