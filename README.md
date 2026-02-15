# युक्ति (Yukti) Programming Language

![Yukti Logo](yukti-logo.png)

**Version:** 1.0  
**Status:** Specification Phase  
**License:** Proprietary - All Rights Reserved

## Overview

Yukti (युक्ति) is a modern, statically-typed, compiled programming language that draws inspiration from Indian philosophical traditions while incorporating contemporary programming paradigms. The name 'Yukti' means "strategy," "method," or "logical reasoning" in Sanskrit.

## Philosophy

Yukti's design is rooted in Indian philosophical concepts:

- **Dharma (धर्म)** - Responsibility through ownership and RAII
- **Karma (कर्म)** - Actions and consequences in concurrency
- **Satya (सत्य)** - Truth through strong typing and correctness
- **Samyama (संयम)** - Discipline through immutability by default

## Key Features

✨ **Static Type System** with type inference  
🔒 **Memory Safety** through ownership and borrowing  
⚡ **Zero-Cost Abstractions** for performance  
🎭 **Pattern Matching** and algebraic data types  
🔄 **Karma-based Concurrency** (actor model)  
🛠️ **Compile-time Metaprogramming** (Mantra system)  
🔗 **C/C++ Interoperability** through FFI  

## Quick Start

### Hello World

```yukti
prakriya main() -> Shunya {
    likho("Namaste, Vishwa!");
}
```

### Variables

```yukti
// Immutable by default
sthir x: Ank32 = 42;

// Mutable when needed
parivartan count: Ank32 = 0;
count = count + 1;
```

### Functions

```yukti
prakriya jod(a: Ank32, b: Ank32) -> Ank32 {
    vapasi a + b;
}

// Pure functions
pavitra prakriya square(n: Ank32) -> Ank32 {
    vapasi n * n;
}
```

### Data Structures

```yukti
sangrah Vyakti {
    naam: Shabd,
    aayu: Ank32,
}

prakar Parinam<T, E> {
    Safal(T),
    Asafal(E),
}
```

### Pattern Matching

```yukti
milaan number {
    0 => "shunya",
    1 => "ek",
    2..=10 => "kuch",
    _ => "bahut"
}
```

### Concurrency (Karma)

```yukti
karma prakriya fetch_data() -> Shabd {
    // Async operation
    vapasi "Data fetched";
}

prakriya main() -> Shunya {
    sthir result = pratiksha fetch_data();
    likho(result);
}
```

## Keywords Reference

| Yukti | Sanskrit/Hindi | English | Purpose |
|-------|----------------|---------|---------|
| prakriya | प्रक्रिया | function | Function declaration |
| sthir | स्थिर | const | Immutable binding |
| parivartan | परिवर्तन | mut/var | Mutable binding |
| yadi | यदि | if | Conditional |
| anyatha | अन्यथा | else | Alternative branch |
| chakra | चक्र | loop | Iteration |
| milaan | मिलान | match | Pattern matching |
| sangrah | संग्रह | struct | Data structure |
| prakar | प्रकार | enum | Enumeration |
| karma | कर्म | async | Concurrent task |
| pratiksha | प्रतीक्षा | await | Wait for completion |
| vapasi | वापसी | return | Return value |

## Type System

### Primitive Types

- **Shunya** - Unit/void type
- **Satya** - Boolean (satya/asatya)
- **Ank8, Ank16, Ank32, Ank64** - Signed integers
- **Uank8, Uank16, Uank32, Uank64** - Unsigned integers
- **Dashamlav32, Dashamlav64** - Floating point
- **Akshar** - Unicode character
- **Shabd** - UTF-8 string

### Advanced Types

- **Sambhavya<T>** - Optional (Some/Kuch or None/Nahin)
- **Parinam<T, E>** - Result (Success/Safal or Failure/Asafal)
- **Suchi<T>** - Dynamic array
- **Muth<K, V>** - Hash map

## Memory Management

### Ownership (Swamitva)

```yukti
sthir s1 = "hello".roop();
sthir s2 = s1;  // s1 is moved to s2
// s1 is no longer valid
```

### Borrowing (Udhar)

```yukti
// Immutable borrow
prakriya lambai(s: &Shabd) -> Ank32 {
    vapasi s.lambai();
}

// Mutable borrow
prakriya jodo(s: &parivartan Shabd, text: Shabd) {
    s.jodo(text);
}
```

## Standard Library (Shastra)

- **shastra::io** - Input/output operations
- **shastra::sangrah** - Collections (Suchi, Muth, Samuchay)
- **shastra::sankhya** - Mathematical operations
- **shastra::samay** - Time and duration
- **shastra::file** - File system operations
- **shastra::jaal** - Networking
- **shastra::karma** - Concurrency primitives

## Example Programs

See the `examples/` directory for complete examples:

- `hello.yukti` - Hello World
- `structures.yukti` - Data structures and methods
- `concurrency.yukti` - Async/await and message passing
- `error_handling.yukti` - Result types and error propagation

## Build System (Nirman)

Projects use the Nirman (निर्माण) build system:

```toml
# Nirman.toml
[package]
naam = "my-project"
version = "0.1.0"
authors = ["Your Name"]

[dependencies]
shastra = "1.0"
```

Build commands:
```bash
nirman build      # Compile project
nirman run        # Compile and run
nirman test       # Run tests
nirman clean      # Clean build artifacts
```

## Compiler (yuktic)

Compilation pipeline:
1. Lexical Analysis
2. Parsing (AST generation)
3. Semantic Analysis (type & borrow checking)
4. MIR Generation
5. Optimization
6. LLVM IR Generation
7. Linking

## Development Roadmap

### Phase 1 (Months 1-12)
- [ ] Core language specification
- [ ] Lexer and parser
- [ ] Basic type system
- [ ] Compiler frontend

### Phase 2 (Months 13-24)
- [ ] Ownership and borrowing system
- [ ] LLVM backend integration
- [ ] Memory management implementation
- [ ] Basic standard library

### Phase 3 (Months 25-36)
- [ ] Full standard library
- [ ] Build system (Nirman)
- [ ] Documentation and tutorials
- [ ] Community tools (debugger, formatter)

### Phase 4 (Ongoing)
- [ ] Package registry
- [ ] IDE support
- [ ] Community growth
- [ ] Ecosystem development

## Philosophy Quotes

> "युक्तियुक्तं वचो ग्राह्यं बालादपि शुकादपि"  
> "Accept the logical, even from a child or a parrot"  
> — Sanskrit Subhashita

This principle guides Yukti's design: embrace good ideas regardless of their source, and let logic and reason prevail.

## Comparison with Other Languages

| Feature | Yukti | Rust | C++ | Go |
|---------|-------|------|-----|-----|
| Memory Safety | ✅ | ✅ | ❌ | ✅ |
| Zero-cost Abstractions | ✅ | ✅ | ✅ | ❌ |
| Ownership System | ✅ | ✅ | ❌ | ❌ |
| Pattern Matching | ✅ | ✅ | ✅ (C++17+) | ❌ |
| Async/Await | ✅ | ✅ | ✅ (C++20) | ✅ |
| Cultural Theme | 🇮🇳 Indian | 🦀 Crab | ➕ Plus | 🐹 Gopher |

## License

This project is proprietary software. All rights reserved. See LICENSE file for details.

This is original work and may not be distributed, modified, or used commercially without explicit permission.

## Project Status

**Current Phase:** Specification and Design  
**License:** Proprietary - All Rights Reserved  
**Contributions:** Not accepting external contributions at this time

## Inspiration

Yukti draws inspiration from:
- **Rust** - Ownership system and memory safety
- **C++** - Performance and systems programming
- **Haskell** - Type system and functional programming
- **Go** - Simplicity and concurrency
- **Sanskrit & Indian Philosophy** - Naming and design principles

## Why Yukti?

1. **Cultural Identity** - Programming with Indian concepts and terminology
2. **Modern Safety** - Memory safety without garbage collection
3. **Performance** - Zero-cost abstractions for systems programming
4. **Expressiveness** - Rich type system and pattern matching
5. **Community** - Built with and for the Indian developer community

## License

**Proprietary License - All Rights Reserved**

Copyright (c) 2026 [Your Name]

This software and associated documentation files are proprietary and confidential. No part of this project may be reproduced, distributed, or transmitted in any form without prior written permission from the copyright holder.

See the LICENSE file for complete terms and conditions.

## Acknowledgments

Special thanks to:
- The Rust team for pioneering safe systems programming
- Bjarne Stroustrup for showing how to evolve a language
- The Indian developer community for inspiration
- All contributors and early adopters

---

**धन्यवाद (Dhanyavaad) - Thank you for your interest in Yukti!**

"Code with wisdom, build with purpose."
