# Yukti Compiler Architecture (yuktic)

## Overview

The Yukti compiler (yuktic) is designed as a modern, multi-pass compiler that transforms Yukti source code into native machine code via LLVM. This document describes the architecture, components, and data flow through the compiler.

## High-Level Architecture

```
Source Code (.yukti)
        ↓
   [Lexer/Scanner]
        ↓
     Tokens
        ↓
     [Parser]
        ↓
   AST (Abstract Syntax Tree)
        ↓
  [Name Resolution]
        ↓
   Resolved AST
        ↓
  [Type Checker]
        ↓
   Typed AST
        ↓
  [Borrow Checker]
        ↓
   Validated AST
        ↓
  [HIR Lowering]
        ↓
   HIR (High-level IR)
        ↓
  [MIR Generation]
        ↓
   MIR (Mid-level IR)
        ↓
  [Optimization Passes]
        ↓
   Optimized MIR
        ↓
  [LLVM IR Generation]
        ↓
   LLVM IR
        ↓
  [LLVM Backend]
        ↓
   Machine Code
        ↓
     [Linker]
        ↓
   Executable
```

## Compiler Phases

### 1. Lexical Analysis (Lexer)

**Purpose:** Transform source text into a stream of tokens

**Input:** Raw source code string  
**Output:** Token stream

**Components:**
- Character stream reader
- Token classifier
- Unicode support for Devanagari
- Error reporting for invalid characters

**Token Types:**
```rust
enum TokenKind {
    // Keywords
    Prakriya, Sthir, Parivartan, Yadi, Anyatha,
    Chakra, Pratyek, Milaan, Sangrah, Prakar,
    // Literals
    IntLiteral(i64),
    FloatLiteral(f64),
    StringLiteral(String),
    CharLiteral(char),
    // Identifiers
    Identifier(String),
    // Operators
    Plus, Minus, Star, Slash, Percent,
    EqEq, NotEq, Lt, Gt, Le, Ge,
    AndAnd, OrOr, Not,
    // Delimiters
    LParen, RParen, LBrace, RBrace, LBracket, RBracket,
    Semi, Comma, Colon, Dot, Arrow, FatArrow,
    // Special
    Eof, Error(String),
}
```

**Error Handling:**
- Unexpected characters
- Unclosed string literals
- Invalid number formats

### 2. Parsing

**Purpose:** Build Abstract Syntax Tree from tokens

**Input:** Token stream  
**Output:** AST (Abstract Syntax Tree)

**Parser Type:** Recursive descent with operator precedence

**AST Node Types:**
```rust
enum Expr {
    Literal(Literal),
    Identifier(String),
    Binary { op: BinOp, left: Box<Expr>, right: Box<Expr> },
    Unary { op: UnaryOp, expr: Box<Expr> },
    Call { func: Box<Expr>, args: Vec<Expr> },
    MethodCall { receiver: Box<Expr>, method: String, args: Vec<Expr> },
    Field { receiver: Box<Expr>, field: String },
    Index { array: Box<Expr>, index: Box<Expr> },
    If { cond: Box<Expr>, then: Block, else_: Option<Box<Expr>> },
    Match { expr: Box<Expr>, arms: Vec<MatchArm> },
    Loop { body: Block },
    While { cond: Box<Expr>, body: Block },
    For { pat: Pattern, iter: Box<Expr>, body: Block },
    Block(Block),
    Return(Option<Box<Expr>>),
    Break(Option<Box<Expr>>),
    Continue,
    Async(Block),
    Await(Box<Expr>),
}

enum Item {
    Function(Function),
    Struct(Struct),
    Enum(Enum),
    Trait(Trait),
    Impl(Impl),
    Const(Const),
}

struct Function {
    name: String,
    generics: Vec<GenericParam>,
    params: Vec<Param>,
    return_type: Option<Type>,
    body: Block,
    is_async: bool,
    is_pure: bool,
}
```

**Error Recovery:**
- Synchronization on statement boundaries
- Error production rules
- Multiple error reporting

### 3. Name Resolution

**Purpose:** Resolve all identifiers to their definitions

**Input:** AST  
**Output:** Resolved AST with symbol table

**Components:**
- Symbol table builder
- Scope stack
- Module resolver
- Import/export handler

**Resolution Algorithm:**
1. Build symbol table for items
2. Resolve imports
3. Walk AST and resolve names
4. Check for undefined variables
5. Handle shadowing correctly

**Symbol Table:**
```rust
struct SymbolTable {
    scopes: Vec<Scope>,
    definitions: HashMap<DefId, Definition>,
}

struct Scope {
    symbols: HashMap<String, DefId>,
    parent: Option<ScopeId>,
}

enum Definition {
    Function(FunctionDef),
    Type(TypeDef),
    Variable(VarDef),
    Const(ConstDef),
}
```

### 4. Type Checking

**Purpose:** Ensure type correctness throughout the program

**Input:** Resolved AST  
**Output:** Typed AST

**Type System Features:**
- Hindley-Milner type inference
- Generic types with bounds
- Trait resolution
- Associated types
- Higher-rank types

**Type Checker Components:**
- Type inference engine
- Unification algorithm
- Trait solver
- Type error reporting

**Type Representation:**
```rust
enum Type {
    Primitive(PrimitiveType),
    Reference { mutable: bool, lifetime: Lifetime, inner: Box<Type> },
    Tuple(Vec<Type>),
    Array { element: Box<Type>, size: Option<usize> },
    Function { params: Vec<Type>, return_type: Box<Type> },
    Generic { name: String, bounds: Vec<TraitBound> },
    Associated { trait_: TraitId, type_name: String },
    UserDefined(DefId),
}
```

**Type Inference:**
- Generate constraints from expressions
- Solve constraints via unification
- Propagate types throughout AST
- Report type errors with context

### 5. Borrow Checker

**Purpose:** Ensure memory safety through ownership and borrowing rules

**Input:** Typed AST  
**Output:** Validated AST

**Checks:**
1. Each value has exactly one owner
2. Borrowing rules are followed
3. Lifetimes are valid
4. No use-after-move errors
5. No use-after-free errors

**Algorithm:**
- Control flow graph construction
- Liveness analysis
- Loan checking
- Lifetime inference

**Data Structures:**
```rust
struct BorrowInfo {
    loans: HashMap<Place, Loan>,
    moves: HashMap<Place, MoveInfo>,
    initializations: HashSet<Place>,
}

struct Loan {
    kind: LoanKind, // Shared or Mutable
    lifetime: Lifetime,
    borrowed_place: Place,
}

enum Place {
    Local(LocalId),
    Field { base: Box<Place>, field: FieldId },
    Index { base: Box<Place>, index: LocalId },
    Deref(Box<Place>),
}
```

### 6. HIR (High-level Intermediate Representation)

**Purpose:** Simplified AST for easier analysis

**Transformations:**
- Desugar syntax
- Normalize control flow
- Make implicit operations explicit
- Remove syntactic sugar

**HIR Features:**
- Simpler structure than AST
- Explicit type information
- Normalized expressions
- Control flow graph

### 7. MIR (Mid-level Intermediate Representation)

**Purpose:** Low-level representation for optimization

**Input:** HIR  
**Output:** MIR

**MIR Structure:**
- Control Flow Graph (CFG)
- Basic blocks
- Statements and terminators
- Explicit temporaries

```rust
struct MirBody {
    basic_blocks: Vec<BasicBlock>,
    local_decls: Vec<LocalDecl>,
}

struct BasicBlock {
    statements: Vec<Statement>,
    terminator: Terminator,
}

enum Statement {
    Assign { place: Place, value: Rvalue },
    StorageLive(Local),
    StorageDead(Local),
    Nop,
}

enum Terminator {
    Goto { target: BasicBlockId },
    SwitchInt { discr: Operand, targets: Vec<(u128, BasicBlockId)> },
    Return,
    Unreachable,
    Call { func: Operand, args: Vec<Operand>, destination: Place, target: BasicBlockId },
}
```

### 8. Optimization Passes

**Purpose:** Improve performance and code size

**MIR Optimizations:**
1. Dead code elimination
2. Constant propagation
3. Constant folding
4. Common subexpression elimination
5. Function inlining
6. Loop unrolling
7. Strength reduction
8. Copy propagation

**Pass Manager:**
```rust
struct OptimizationPipeline {
    passes: Vec<Box<dyn MirPass>>,
}

trait MirPass {
    fn run_pass(&self, body: &mut MirBody);
}
```

### 9. LLVM IR Generation

**Purpose:** Translate MIR to LLVM IR

**Input:** Optimized MIR  
**Output:** LLVM IR

**Translation:**
- Map Yukti types to LLVM types
- Generate LLVM functions
- Handle calling conventions
- Generate debug information

**LLVM Integration:**
```rust
struct CodegenContext {
    llvm_context: LLVMContext,
    llvm_module: Module,
    builder: Builder,
    type_cache: HashMap<TypeId, LLVMType>,
}
```

### 10. Linking

**Purpose:** Combine object files into executable

**Linker Operations:**
- Link compiled objects
- Resolve external symbols
- Handle dynamic libraries
- Generate final executable

## Compiler Driver

**Main Entry Point:**
```rust
fn main() {
    let args = parse_args();
    let compiler = Compiler::new(args);
    
    match compiler.compile() {
        Ok(_) => println!("Compilation successful"),
        Err(errors) => {
            for error in errors {
                eprintln!("{}", error);
            }
            std::process::exit(1);
        }
    }
}
```

**Compiler Structure:**
```rust
struct Compiler {
    session: Session,
    source_map: SourceMap,
    error_handler: ErrorHandler,
}

impl Compiler {
    fn compile(&self) -> Result<(), Vec<CompileError>> {
        let tokens = self.lex()?;
        let ast = self.parse(tokens)?;
        let resolved_ast = self.resolve_names(ast)?;
        let typed_ast = self.type_check(resolved_ast)?;
        self.borrow_check(&typed_ast)?;
        let hir = self.lower_to_hir(typed_ast)?;
        let mir = self.generate_mir(hir)?;
        let optimized_mir = self.optimize(mir)?;
        let llvm_ir = self.generate_llvm_ir(optimized_mir)?;
        self.emit_object_file(llvm_ir)?;
        self.link()?;
        Ok(())
    }
}
```

## Error Handling

**Error Categories:**
1. Syntax errors
2. Name resolution errors
3. Type errors
4. Borrow checking errors
5. Internal compiler errors (ICE)

**Error Reporting:**
```rust
struct CompileError {
    kind: ErrorKind,
    span: Span,
    message: String,
    notes: Vec<String>,
    suggestions: Vec<Suggestion>,
}

struct Suggestion {
    message: String,
    span: Span,
    replacement: String,
}
```

## Incremental Compilation

**Features:**
- Dependency tracking
- Query-based compilation
- Caching intermediate results
- Minimal recompilation

**Query System:**
```rust
trait QueryProvider {
    fn parse_source(&self, file_id: FileId) -> Arc<AST>;
    fn type_check(&self, item_id: ItemId) -> TypeCheckResult;
    fn generate_mir(&self, item_id: ItemId) -> Arc<MirBody>;
}
```

## Parallel Compilation

**Strategies:**
- Parallel parsing of multiple files
- Parallel type checking of independent items
- Parallel code generation
- Thread pool for compilation tasks

## Debugger Support

**Debug Information Generation:**
- DWARF debug info
- Source maps
- Variable locations
- Type information

## Performance Considerations

**Optimizations:**
- Arena allocation for AST nodes
- String interning for identifiers
- Efficient symbol tables
- Lazy evaluation where possible

**Benchmarks:**
- Compilation speed
- Memory usage
- Generated code quality

## Testing Strategy

**Test Types:**
1. Unit tests for each component
2. Integration tests for compiler pipeline
3. UI tests for error messages
4. Performance regression tests
5. Fuzz testing

## Implementation Timeline

### Phase 1: Foundation (Months 1-3)
- Lexer implementation
- Parser for basic constructs
- Simple AST
- Basic error reporting

### Phase 2: Type System (Months 4-6)
- Name resolution
- Type inference
- Basic trait system
- Type error reporting

### Phase 3: Safety (Months 7-9)
- Ownership tracking
- Borrow checker
- Lifetime inference
- Safety error messages

### Phase 4: Code Generation (Months 10-12)
- HIR and MIR design
- LLVM integration
- Basic optimizations
- Executable generation

### Phase 5: Polish (Months 13+)
- Advanced optimizations
- Incremental compilation
- Better error messages
- Performance tuning

## Conclusion

The Yukti compiler is designed to be:
- **Correct**: Strong guarantees through static analysis
- **Fast**: Efficient compilation through parallelization
- **Helpful**: Excellent error messages and suggestions
- **Maintainable**: Clean architecture and extensive testing

This architecture provides a solid foundation for implementing a production-quality compiler for the Yukti programming language.
