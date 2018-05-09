---
layout: post
title: "overview for compiler "
description: "overview for compiler"
categories: [Compiler]
tags: [Compiler]
redirect_from:
  - /2018/05/09/
---
## overview for compiler 

analysis part: breaks the source program into constituent pieces and impose a grammatical structure on them and create symbol table  ---> **front end**
synthesis part: constructs the desired target program from the intermediate representation and the information in the symbol table.  --> **back end**

### part 1. lexical analysis / scanning
scanner reads the character streams making up the source program and groups the characters into meaningful sequences called **lexemes**. 
For each lexeme, the lexical analyzer produces a **token** of the form
> <token-name, attribute-value>

variable names become **id**--> create symbol table
### part 2. syntax analysis
This phase is called **syntax analysis** or **parsing**. The parser uses the first components of the tokens produced by the lexical analyzer to create a tree-like intermediate representation that depicts the grammatical structure of the token stream.  --> **syntax tree**

### part 3. semantic analysis
The semantic analyzer use the syntax tree and symbol table to check the source program for semantic consistency with the language definition. It also gathers type information and saves it in either the syntax tree or the symbol table, for subsequent use during intermediate-code generation phase. 
**type checking** happens in this phase. 
### part 4. intermediate code generation
use syntax tree to create an explicit low-level or machine-like intermediate representation. **three-address code**

### part 5. code optimization
The machine-independent code-optimization phase attempts to improve the intermediate code so that better target code will result. 

### part 6. code generation
The code generator takes intermediate code as input and maps it into the target language. The intermediate instructions are translated into sequence of machine instructions that perform the same task. 