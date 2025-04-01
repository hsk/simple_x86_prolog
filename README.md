# simple_x86_prolog

A minimalist yet elegant x86_64 native compiler crafted in Prolog. This experimental project transforms a simple `while`-based language into Static Single Assignment (SSA) form, performs liveness analysis, and generates executable code—all in roughly 285 lines of concise Prolog brilliance.

- **Target**: x86_64 architecture
- **Core Features**: SSA transformation, liveness analysis, x86_64 code generation
- **Key Files**: `ssa.pl`, `live.pl`, `x86_gen.pl`
- **Author**: hsk (If it goes viral, feel free to laugh—or applaud)

## Overview

This toy language compiler weighs in at 285 lines across multiple modules, delivering a surprisingly capable pipeline from source to executable:


```bash
$ ls *.pl | grep -v all.pl | xargs wc
    47   132  3075 genAmd64.pl
    26    63  1451 genCode.pl
    36    82  1958 graph.pl
    27    62  1527 graphRegAlloc.pl
    29    70  1778 linearScanRegAlloc.pl
    37    78  2048 liveness.pl
    26    71  1387 main.pl
    23    59  1222 memAlloc.pl
    34   163  1062 syntax.pl
   285   780 15508 total
```

## Syntax

The compiler supports a lightweight language with the following grammar:

- **Expressions (`e`)**: `i | x | e + e | e - e | x = e | x(e1,...,en)`
- **Statements (`s`)**: `return(e) | if(e,s*,s*) | while(e,s*) | e`
- **Definitions (`d`)**: `x(x1,...,xn)=s*`
- **Program (`p`)**: `d*`

Intermediate representations include:
- **Abstract Operations (`o`, `ae`, `as`, `af`)**: Supporting `addq`, `subq`, and control flow.
- **Concrete Code (`cr`, `l`, `cc`, `cf`)**: Register-based operations and branching.
- **Register Representation (`ri`, `rr`, `rc`, `rf`)**: Final x86_64-ready instructions.

## Features

- **Simple Memory Allocation**: Efficiently assigns memory addresses for variables.
- **Linear Scan Register Allocation**: Implements fast, practical register assignment with spilling, using a simple memory allocation fallback.
- **Graph Coloring Register Allocation**: Leverages the Welsh & Powell algorithm for optimal register use, with spilling handled similarly.

## Installation

Ensure you have the prerequisites:

```bash
apt install swi-prolog gcc
```

## Usage

Compile and run your program effortlessly:

```bash
$ swipl main.pl src.mc
$ gcc a.s -static lib/lib.c -o a.out
$ ./a.out
54321
55
```

Or use shortcuts:

```bash
$ make
```

To run the test suite:

```bash
$ make test
```

## Example Source Program

Here’s a sample [src.mc](src.mc) showcasing the language’s capabilities:

```prolog
% src.mc
main()=[
    if(0,[
        a=0
    ],[
        a=50000+5000-1000
    ]),
    printInt(a+add(300,20,1)),
    printInt(sum(10)),
    return(0)
].
sum(n)=[
    if(n,[return(sum(n-1)+n)],[]),
    return(n)
].
add(a,b,c)=[
    return(a+b+c)
].
add2(a,b)=[
    return(a+b)
].
```

## One Source compiler

For minimalists, [all.pl](all.pl) offers a 128-line, all-in-one memory allocation compiler:

```bash
$ swipl all.pl src.mc && gcc -static a.s lib/lib.c && ./a.out
54321
55
```

## Compilation Pipeline

The compilation process is elegantly modular:

- **Parsing (`*.mc`)**: Reads Prolog-style terms and constructs an AST ([main.pl](main.pl), 26 lines).
- **Code Generation**: Transforms the AST into internal basic block representations ([genCode.pl](genCode.pl), 26 lines).
- **Memory Allocation**: Assigns memory addresses to variables ([memAlloc.pl](memAlloc.pl), 23 lines).
- **Assembly Output**: Generates x86_64 assembly into a `.s` file ([genAmd64.pl](genAmd64.pl), 47 lines).
- **Syntax Validation**: Ensures correctness across language stages ([syntax.pl](syntax.pl), 34 lines).

Total: 122 lines for the core pipeline.

## Optimizations
Advanced register allocation techniques enhance performance:
- **Liveness Analysis**: A dataflow analysis computing variable lifetimes within basic blocks, creating a shadow tree ([liveness.pl](liveness.pl), 78 lines).
- **Linear Scan Allocation**: Fast register assignment with spilling support ([linearScanRegAlloc.pl](linearScanRegAlloc.pl), 29 lines).
- **Graph Coloring Allocation**: Uses the Welsh & Powell algorithm for precise register optimization ([graph.pl](graph.pl), 82 lines; [graphRegAlloc.pl](graphRegAlloc.pl), 27 lines).

**Grand total**: 285 lines of pure Prolog ingenuity.
