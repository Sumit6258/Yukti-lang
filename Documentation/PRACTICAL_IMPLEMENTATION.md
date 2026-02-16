# Practical Compiler Implementation Guide for Yukti

## Overview

This guide provides a concrete, step-by-step approach to implementing the Yukti compiler. We'll build a minimal viable compiler that can compile simple Yukti programs to executable binaries.

---

## Part 1: Prerequisites & Setup

### Required Knowledge
- **C++ or Rust** (intermediate level)
- **Basic compiler theory** (lexing, parsing, AST)
- **LLVM basics** (or willingness to learn)

### Tools You'll Need

```bash
# 1. Install LLVM
# Ubuntu/Debian
sudo apt-get install llvm-14 llvm-14-dev clang-14

# macOS
brew install llvm

# Windows
# Download from llvm.org

# 2. Install Build Tools
# Linux
sudo apt-get install cmake build-essential

# macOS
xcode-select --install
brew install cmake

# Windows
# Install Visual Studio 2022 with C++ tools
```

---

## Part 2: Project Structure

Create this directory structure:

```
yukti-compiler/
├── CMakeLists.txt           # Build configuration
├── src/
│   ├── main.cpp             # Entry point
│   ├── lexer/
│   │   ├── lexer.h
│   │   ├── lexer.cpp
│   │   └── token.h
│   ├── parser/
│   │   ├── parser.h
│   │   ├── parser.cpp
│   │   └── ast.h
│   ├── semantic/
│   │   ├── type_checker.h
│   │   └── type_checker.cpp
│   └── codegen/
│       ├── codegen.h
│       └── codegen.cpp
├── tests/
│   └── test_programs/
│       ├── hello.yukti
│       └── simple.yukti
└── runtime/
    └── runtime.c            # Runtime library
```

---

## Part 3: Minimal Implementation (Phase 1)

### Step 1: Token Definition (token.h)

```cpp
// src/lexer/token.h
#ifndef TOKEN_H
#define TOKEN_H

#include <string>
#include <variant>

enum class TokenType {
    // Keywords
    PRAKRIYA,      // function
    STHIR,         // const
    PARIVARTAN,    // mut
    YADI,          // if
    ANYATHA,       // else
    VAPASI,        // return
    LIKHO,         // print
    
    // Types
    ANK32,         // i32
    SHABD,         // string
    SHUNYA,        // void
    
    // Literals
    INTEGER,
    STRING,
    
    // Operators
    PLUS, MINUS, STAR, SLASH,
    EQ, EQEQ, LT, GT,
    LPAREN, RPAREN, LBRACE, RBRACE,
    SEMICOLON, COMMA, ARROW,
    
    // Special
    IDENTIFIER,
    END_OF_FILE,
    ERROR
};

struct Token {
    TokenType type;
    std::string lexeme;
    std::variant<int, std::string> value;
    int line;
    int column;
};

#endif
```

### Step 2: Lexer Implementation (lexer.cpp)

```cpp
// src/lexer/lexer.cpp
#include "lexer.h"
#include <cctype>
#include <unordered_map>

class Lexer {
private:
    std::string source;
    size_t current = 0;
    size_t line = 1;
    size_t column = 1;
    
    static const std::unordered_map<std::string, TokenType> keywords;
    
    char peek() { return current < source.length() ? source[current] : '\0'; }
    char advance() { 
        column++; 
        return source[current++]; 
    }
    
    void skipWhitespace() {
        while (std::isspace(peek())) {
            if (peek() == '\n') {
                line++;
                column = 0;
            }
            advance();
        }
    }
    
    Token makeToken(TokenType type, const std::string& lexeme) {
        return Token{type, lexeme, {}, (int)line, (int)column};
    }
    
    Token scanNumber() {
        std::string num;
        while (std::isdigit(peek())) {
            num += advance();
        }
        return Token{TokenType::INTEGER, num, std::stoi(num), (int)line, (int)column};
    }
    
    Token scanString() {
        advance(); // Skip opening quote
        std::string str;
        while (peek() != '"' && peek() != '\0') {
            str += advance();
        }
        advance(); // Skip closing quote
        return Token{TokenType::STRING, str, str, (int)line, (int)column};
    }
    
    Token scanIdentifier() {
        std::string id;
        while (std::isalnum(peek()) || peek() == '_') {
            id += advance();
        }
        
        auto it = keywords.find(id);
        if (it != keywords.end()) {
            return makeToken(it->second, id);
        }
        return Token{TokenType::IDENTIFIER, id, id, (int)line, (int)column};
    }

public:
    Lexer(const std::string& src) : source(src) {}
    
    std::vector<Token> tokenize() {
        std::vector<Token> tokens;
        
        while (current < source.length()) {
            skipWhitespace();
            if (current >= source.length()) break;
            
            char c = peek();
            
            // Single character tokens
            if (c == '(') { tokens.push_back(makeToken(TokenType::LPAREN, "(")); advance(); }
            else if (c == ')') { tokens.push_back(makeToken(TokenType::RPAREN, ")")); advance(); }
            else if (c == '{') { tokens.push_back(makeToken(TokenType::LBRACE, "{")); advance(); }
            else if (c == '}') { tokens.push_back(makeToken(TokenType::RBRACE, "}")); advance(); }
            else if (c == ';') { tokens.push_back(makeToken(TokenType::SEMICOLON, ";")); advance(); }
            else if (c == ',') { tokens.push_back(makeToken(TokenType::COMMA, ",")); advance(); }
            else if (c == '+') { tokens.push_back(makeToken(TokenType::PLUS, "+")); advance(); }
            else if (c == '*') { tokens.push_back(makeToken(TokenType::STAR, "*")); advance(); }
            else if (c == '/') { tokens.push_back(makeToken(TokenType::SLASH, "/")); advance(); }
            
            // Multi-character tokens
            else if (c == '-') {
                advance();
                if (peek() == '>') {
                    advance();
                    tokens.push_back(makeToken(TokenType::ARROW, "->"));
                } else {
                    tokens.push_back(makeToken(TokenType::MINUS, "-"));
                }
            }
            else if (c == '=') {
                advance();
                if (peek() == '=') {
                    advance();
                    tokens.push_back(makeToken(TokenType::EQEQ, "=="));
                } else {
                    tokens.push_back(makeToken(TokenType::EQ, "="));
                }
            }
            
            // Literals
            else if (std::isdigit(c)) {
                tokens.push_back(scanNumber());
            }
            else if (c == '"') {
                tokens.push_back(scanString());
            }
            
            // Identifiers and keywords
            else if (std::isalpha(c) || c == '_') {
                tokens.push_back(scanIdentifier());
            }
            
            else {
                advance(); // Skip unknown character
            }
        }
        
        tokens.push_back(makeToken(TokenType::END_OF_FILE, ""));
        return tokens;
    }
};

// Keyword mapping
const std::unordered_map<std::string, TokenType> Lexer::keywords = {
    {"prakriya", TokenType::PRAKRIYA},
    {"sthir", TokenType::STHIR},
    {"parivartan", TokenType::PARIVARTAN},
    {"yadi", TokenType::YADI},
    {"anyatha", TokenType::ANYATHA},
    {"vapasi", TokenType::VAPASI},
    {"likho", TokenType::LIKHO},
    {"Ank32", TokenType::ANK32},
    {"Shabd", TokenType::SHABD},
    {"Shunya", TokenType::SHUNYA},
};
```

### Step 3: Abstract Syntax Tree (ast.h)

```cpp
// src/parser/ast.h
#ifndef AST_H
#define AST_H

#include <string>
#include <vector>
#include <memory>
#include <variant>

// Forward declarations
struct Expr;
struct Stmt;

using ExprPtr = std::unique_ptr<Expr>;
using StmtPtr = std::unique_ptr<Stmt>;

// Expression types
struct IntegerLiteral {
    int value;
};

struct StringLiteral {
    std::string value;
};

struct Identifier {
    std::string name;
};

struct BinaryOp {
    ExprPtr left;
    std::string op;
    ExprPtr right;
};

struct CallExpr {
    std::string function;
    std::vector<ExprPtr> arguments;
};

struct Expr {
    std::variant<IntegerLiteral, StringLiteral, Identifier, BinaryOp, CallExpr> value;
};

// Statement types
struct VarDecl {
    std::string name;
    std::string type;
    ExprPtr initializer;
    bool is_mutable;
};

struct ReturnStmt {
    ExprPtr value;
};

struct ExprStmt {
    ExprPtr expression;
};

struct Stmt {
    std::variant<VarDecl, ReturnStmt, ExprStmt> value;
};

// Function definition
struct Function {
    std::string name;
    std::vector<std::pair<std::string, std::string>> parameters; // (name, type)
    std::string return_type;
    std::vector<StmtPtr> body;
};

// Program (collection of functions)
struct Program {
    std::vector<Function> functions;
};

#endif
```

### Step 4: Parser (parser.cpp)

```cpp
// src/parser/parser.cpp
#include "parser.h"
#include <stdexcept>

class Parser {
private:
    std::vector<Token> tokens;
    size_t current = 0;
    
    Token peek() { return tokens[current]; }
    Token previous() { return tokens[current - 1]; }
    bool isAtEnd() { return peek().type == TokenType::END_OF_FILE; }
    
    Token advance() {
        if (!isAtEnd()) current++;
        return previous();
    }
    
    bool check(TokenType type) {
        if (isAtEnd()) return false;
        return peek().type == type;
    }
    
    bool match(TokenType type) {
        if (check(type)) {
            advance();
            return true;
        }
        return false;
    }
    
    void expect(TokenType type, const std::string& message) {
        if (!match(type)) {
            throw std::runtime_error(message);
        }
    }
    
    ExprPtr parsePrimary() {
        if (match(TokenType::INTEGER)) {
            auto expr = std::make_unique<Expr>();
            expr->value = IntegerLiteral{std::get<int>(previous().value)};
            return expr;
        }
        
        if (match(TokenType::STRING)) {
            auto expr = std::make_unique<Expr>();
            expr->value = StringLiteral{std::get<std::string>(previous().value)};
            return expr;
        }
        
        if (match(TokenType::IDENTIFIER)) {
            std::string name = previous().lexeme;
            
            // Function call
            if (match(TokenType::LPAREN)) {
                std::vector<ExprPtr> args;
                if (!check(TokenType::RPAREN)) {
                    do {
                        args.push_back(parseExpression());
                    } while (match(TokenType::COMMA));
                }
                expect(TokenType::RPAREN, "Expected ')' after arguments");
                
                auto expr = std::make_unique<Expr>();
                expr->value = CallExpr{name, std::move(args)};
                return expr;
            }
            
            // Variable reference
            auto expr = std::make_unique<Expr>();
            expr->value = Identifier{name};
            return expr;
        }
        
        if (match(TokenType::LPAREN)) {
            auto expr = parseExpression();
            expect(TokenType::RPAREN, "Expected ')' after expression");
            return expr;
        }
        
        throw std::runtime_error("Expected expression");
    }
    
    ExprPtr parseMultiplicative() {
        auto expr = parsePrimary();
        
        while (match(TokenType::STAR) || match(TokenType::SLASH)) {
            std::string op = previous().lexeme;
            auto right = parsePrimary();
            
            auto binary = std::make_unique<Expr>();
            binary->value = BinaryOp{std::move(expr), op, std::move(right)};
            expr = std::move(binary);
        }
        
        return expr;
    }
    
    ExprPtr parseAdditive() {
        auto expr = parseMultiplicative();
        
        while (match(TokenType::PLUS) || match(TokenType::MINUS)) {
            std::string op = previous().lexeme;
            auto right = parseMultiplicative();
            
            auto binary = std::make_unique<Expr>();
            binary->value = BinaryOp{std::move(expr), op, std::move(right)};
            expr = std::move(binary);
        }
        
        return expr;
    }
    
    ExprPtr parseExpression() {
        return parseAdditive();
    }
    
    StmtPtr parseStatement() {
        // Variable declaration
        if (match(TokenType::STHIR) || match(TokenType::PARIVARTAN)) {
            bool is_mut = previous().type == TokenType::PARIVARTAN;
            
            expect(TokenType::IDENTIFIER, "Expected variable name");
            std::string name = previous().lexeme;
            
            std::string type;
            if (match(TokenType::ANK32)) type = "Ank32";
            else if (match(TokenType::SHABD)) type = "Shabd";
            
            expect(TokenType::EQ, "Expected '=' in variable declaration");
            auto init = parseExpression();
            expect(TokenType::SEMICOLON, "Expected ';' after statement");
            
            auto stmt = std::make_unique<Stmt>();
            stmt->value = VarDecl{name, type, std::move(init), is_mut};
            return stmt;
        }
        
        // Return statement
        if (match(TokenType::VAPASI)) {
            auto value = parseExpression();
            expect(TokenType::SEMICOLON, "Expected ';' after return");
            
            auto stmt = std::make_unique<Stmt>();
            stmt->value = ReturnStmt{std::move(value)};
            return stmt;
        }
        
        // Expression statement
        auto expr = parseExpression();
        expect(TokenType::SEMICOLON, "Expected ';' after expression");
        
        auto stmt = std::make_unique<Stmt>();
        stmt->value = ExprStmt{std::move(expr)};
        return stmt;
    }
    
    Function parseFunction() {
        expect(TokenType::PRAKRIYA, "Expected 'prakriya'");
        expect(TokenType::IDENTIFIER, "Expected function name");
        std::string name = previous().lexeme;
        
        expect(TokenType::LPAREN, "Expected '(' after function name");
        
        std::vector<std::pair<std::string, std::string>> params;
        if (!check(TokenType::RPAREN)) {
            do {
                expect(TokenType::IDENTIFIER, "Expected parameter name");
                std::string param_name = previous().lexeme;
                
                std::string param_type;
                if (match(TokenType::ANK32)) param_type = "Ank32";
                else if (match(TokenType::SHABD)) param_type = "Shabd";
                
                params.push_back({param_name, param_type});
            } while (match(TokenType::COMMA));
        }
        
        expect(TokenType::RPAREN, "Expected ')' after parameters");
        
        std::string return_type = "Shunya";
        if (match(TokenType::ARROW)) {
            if (match(TokenType::ANK32)) return_type = "Ank32";
            else if (match(TokenType::SHABD)) return_type = "Shabd";
            else if (match(TokenType::SHUNYA)) return_type = "Shunya";
        }
        
        expect(TokenType::LBRACE, "Expected '{' before function body");
        
        std::vector<StmtPtr> body;
        while (!check(TokenType::RBRACE) && !isAtEnd()) {
            body.push_back(parseStatement());
        }
        
        expect(TokenType::RBRACE, "Expected '}' after function body");
        
        return Function{name, params, return_type, std::move(body)};
    }

public:
    Parser(const std::vector<Token>& tokens) : tokens(tokens) {}
    
    Program parse() {
        Program program;
        
        while (!isAtEnd()) {
            program.functions.push_back(parseFunction());
        }
        
        return program;
    }
};
```

### Step 5: LLVM Code Generation (codegen.cpp)

```cpp
// src/codegen/codegen.cpp
#include "codegen.h"
#include <llvm/IR/LLVMContext.h>
#include <llvm/IR/Module.h>
#include <llvm/IR/IRBuilder.h>
#include <llvm/IR/Verifier.h>
#include <llvm/Support/raw_ostream.h>

class CodeGenerator {
private:
    llvm::LLVMContext context;
    llvm::IRBuilder<> builder;
    std::unique_ptr<llvm::Module> module;
    std::map<std::string, llvm::Value*> namedValues;
    
    llvm::Value* codegenExpr(const Expr& expr) {
        if (auto* intLit = std::get_if<IntegerLiteral>(&expr.value)) {
            return llvm::ConstantInt::get(context, llvm::APInt(32, intLit->value));
        }
        
        if (auto* binOp = std::get_if<BinaryOp>(&expr.value)) {
            llvm::Value* L = codegenExpr(*binOp->left);
            llvm::Value* R = codegenExpr(*binOp->right);
            
            if (binOp->op == "+") return builder.CreateAdd(L, R, "addtmp");
            if (binOp->op == "-") return builder.CreateSub(L, R, "subtmp");
            if (binOp->op == "*") return builder.CreateMul(L, R, "multmp");
            if (binOp->op == "/") return builder.CreateSDiv(L, R, "divtmp");
        }
        
        if (auto* id = std::get_if<Identifier>(&expr.value)) {
            llvm::Value* V = namedValues[id->name];
            if (!V) throw std::runtime_error("Unknown variable name");
            return builder.CreateLoad(llvm::Type::getInt32Ty(context), V, id->name);
        }
        
        if (auto* call = std::get_if<CallExpr>(&expr.value)) {
            llvm::Function* CalleeF = module->getFunction(call->function);
            if (!CalleeF) throw std::runtime_error("Unknown function");
            
            std::vector<llvm::Value*> ArgsV;
            for (auto& arg : call->arguments) {
                ArgsV.push_back(codegenExpr(*arg));
            }
            
            return builder.CreateCall(CalleeF, ArgsV, "calltmp");
        }
        
        return nullptr;
    }
    
    void codegenStatement(const Stmt& stmt, llvm::Function* function) {
        if (auto* varDecl = std::get_if<VarDecl>(&stmt.value)) {
            llvm::Value* initVal = codegenExpr(*varDecl->initializer);
            llvm::AllocaInst* alloca = builder.CreateAlloca(
                llvm::Type::getInt32Ty(context), nullptr, varDecl->name
            );
            builder.CreateStore(initVal, alloca);
            namedValues[varDecl->name] = alloca;
        }
        
        if (auto* retStmt = std::get_if<ReturnStmt>(&stmt.value)) {
            llvm::Value* retVal = codegenExpr(*retStmt->value);
            builder.CreateRet(retVal);
        }
        
        if (auto* exprStmt = std::get_if<ExprStmt>(&stmt.value)) {
            codegenExpr(*exprStmt->expression);
        }
    }
    
    llvm::Function* codegenFunction(const Function& func) {
        std::vector<llvm::Type*> Ints(func.parameters.size(), 
                                       llvm::Type::getInt32Ty(context));
        
        llvm::Type* returnType = llvm::Type::getInt32Ty(context);
        if (func.return_type == "Shunya") {
            returnType = llvm::Type::getVoidTy(context);
        }
        
        llvm::FunctionType* FT = llvm::FunctionType::get(returnType, Ints, false);
        llvm::Function* F = llvm::Function::Create(
            FT, llvm::Function::ExternalLinkage, func.name, module.get()
        );
        
        llvm::BasicBlock* BB = llvm::BasicBlock::Create(context, "entry", F);
        builder.SetInsertPoint(BB);
        
        namedValues.clear();
        unsigned Idx = 0;
        for (auto& Arg : F->args()) {
            llvm::AllocaInst* Alloca = builder.CreateAlloca(
                llvm::Type::getInt32Ty(context), nullptr, 
                func.parameters[Idx].first
            );
            builder.CreateStore(&Arg, Alloca);
            namedValues[func.parameters[Idx].first] = Alloca;
            Idx++;
        }
        
        for (auto& stmt : func.body) {
            codegenStatement(*stmt, F);
        }
        
        if (func.return_type == "Shunya") {
            builder.CreateRetVoid();
        }
        
        llvm::verifyFunction(*F);
        return F;
    }

public:
    CodeGenerator() : builder(context) {
        module = std::make_unique<llvm::Module>("yukti", context);
    }
    
    void generate(const Program& program) {
        for (const auto& func : program.functions) {
            codegenFunction(func);
        }
    }
    
    void printIR() {
        module->print(llvm::outs(), nullptr);
    }
    
    void writeObjectFile(const std::string& filename) {
        // Implementation to write object file
        // Would use LLVM's TargetMachine
    }
};
```

### Step 6: Main Entry Point (main.cpp)

```cpp
// src/main.cpp
#include <iostream>
#include <fstream>
#include <sstream>
#include "lexer/lexer.h"
#include "parser/parser.h"
#include "codegen/codegen.h"

int main(int argc, char** argv) {
    if (argc != 2) {
        std::cerr << "Usage: yuktic <file.yukti>\n";
        return 1;
    }
    
    // Read source file
    std::ifstream file(argv[1]);
    if (!file.is_open()) {
        std::cerr << "Error: Could not open file " << argv[1] << "\n";
        return 1;
    }
    
    std::stringstream buffer;
    buffer << file.rdbuf();
    std::string source = buffer.str();
    
    try {
        // Lexical analysis
        Lexer lexer(source);
        auto tokens = lexer.tokenize();
        
        std::cout << "=== TOKENS ===\n";
        for (const auto& token : tokens) {
            std::cout << "  " << token.lexeme << "\n";
        }
        
        // Parsing
        Parser parser(tokens);
        auto program = parser.parse();
        
        std::cout << "\n=== PARSED SUCCESSFULLY ===\n";
        
        // Code generation
        CodeGenerator codegen;
        codegen.generate(program);
        
        std::cout << "\n=== LLVM IR ===\n";
        codegen.printIR();
        
        std::cout << "\n=== COMPILATION SUCCESSFUL ===\n";
        
    } catch (const std::exception& e) {
        std::cerr << "Error: " << e.what() << "\n";
        return 1;
    }
    
    return 0;
}
```

### Step 7: Build Configuration (CMakeLists.txt)

```cmake
cmake_minimum_required(VERSION 3.13)
project(YuktiCompiler)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Find LLVM
find_package(LLVM REQUIRED CONFIG)
include_directories(${LLVM_INCLUDE_DIRS})
add_definitions(${LLVM_DEFINITIONS})

# Compiler sources
set(SOURCES
    src/main.cpp
    src/lexer/lexer.cpp
    src/parser/parser.cpp
    src/codegen/codegen.cpp
)

# Create executable
add_executable(yuktic ${SOURCES})

# Link LLVM libraries
llvm_map_components_to_libnames(llvm_libs support core irreader)
target_link_libraries(yuktic ${llvm_libs})

# Include directories
target_include_directories(yuktic PRIVATE src)
```

---

## Part 4: Building and Testing

### Build the Compiler

```bash
mkdir build
cd build
cmake ..
make

# You should now have the 'yuktic' executable
```

### Test with Simple Program

Create `test.yukti`:
```yukti
prakriya main() -> Ank32 {
    sthir x = 5;
    sthir y = 10;
    vapasi x + y;
}
```

Run:
```bash
./yuktic test.yukti
```

---

## Part 5: What This Gives You

This minimal compiler can:
✅ Lex Yukti source code
✅ Parse into AST
✅ Generate LLVM IR
✅ Handle basic arithmetic
✅ Support functions
✅ Create variables

**What's Missing (for full implementation):**
- Type checking
- Borrow checking
- Standard library
- Full language features
- Optimizations
- Proper error messages

---

## Part 6: Extending the Compiler

To add more features:

1. **Add more tokens** to lexer
2. **Extend AST** nodes
3. **Add parser rules**
4. **Generate corresponding LLVM IR**
5. **Test thoroughly**

Example: Adding if statements
- Add `IfStmt` to AST
- Add parsing in `parseStatement()`
- Generate LLVM conditional branches in codegen

---

## Timeline for Full Implementation

- **Weeks 1-2**: Setup + lexer
- **Weeks 3-4**: Parser
- **Weeks 5-6**: Basic codegen
- **Weeks 7-8**: Type system
- **Weeks 9-12**: More features
- **Months 4-6**: Standard library
- **Months 7-12**: Polish + tools

---

## Next Steps

1. Build this minimal compiler
2. Test with simple programs
3. Add features incrementally
4. Read LLVM tutorial for advanced codegen
5. Study Rust/Swift compilers for inspiration

This gives you a **working foundation** to build upon!
