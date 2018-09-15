---
layout: post
title: "Advanced DB Quert Compilation"
description: "Query Compilation"
categories: [Database]
tags: [Database]
redirect from:
  - /2018/09/15/
---
## Advanced DB Query Compilation
Agenda:
code generation
JIT compilation

When switch to an in-memory DBMS, the only way to increase the throughput is to reduce the number of instructions executed. 
One way to achieve such a reduction in instructions is through **code specialization**. This means generating code that is specific to a particular task in the DBMS. 

### Query Processing Model

- Tuple at a time
- Operator at a time(Materialization model)-> good for OLTP
- vector at a time (batch materialization)

```
where B.val = ? + 1
```
We need to look at the schema to lookup B's val value type. 
Current Tuple -> Query Parameters -> Table Schema
This is an expensive operation. 

Any CPU intensive entity of database can be natively compiled if they have a similar execution pattern on different inputs. 
- access methods
- stored procedures
- operator execution
- predicate evaluation
- logging operation

Benefits
- Attribute types are known **a priori**
Data access function calls can be converted to inline pointer casting
- Predicates are known **a priori**
They can be evaluated using primitive data comparisons
- No function calls in loops
Allows the compiler to efficiently distribute data to registers and increase cache reuse. 

![Alt text](/assets/images/1537027143114.png)

### Code Generation
- Approach 1: Transpilation
Write code that converts a relational query plan into C/C++ and then run it through a conventional compiler to generate native code
- Approach 2: JIT Compilation
Generate an **intermediate representation** of they query that can be quickly compiled into native code

#### Transpilation
For a given query plan, create a C/C++ program
that implements that query’s execution.
→ Bake in all the predicates and type conversions

Use an off-shelf compiler to convert the code into
a shared object, link it to the DBMS process, and
then invoke the exec function.

Relational operators are a useful way to reason about a query but are not the most efficient way to execute it. 
It takes a long time to compile a C/C++ source file into executable code. 

#### Pipelined Operator
![Alt text](/assets/images/1537028381129.png)

### JIT Query Compilation
Compile queries in-memory into native code using
the LLVM toolkit.
Organizes query processing in a way to keep a
tuple in CPU registers for as long as possible.
→ Push-based vs. Pull-based
→ Data Centric vs. Operator Centric

LLVM
Collection of modular and reusable compiler and toolchain technologies
Core component is a low-level programming language(IR) that is similar to assembly. 
Not all of the DBMS components need to written in LLVM IR. 

### Query Compilation Cost
LLVM's compilation time grows super-linearly relative to the query size. 
Solution:**Adaptive Execution**
First generate the LLVM IR for the query. Then execute that IR in an interpreter. Compile the query in the background. 
When the compiled query is ready, seamlessly replace the interpretive execution. 
![Alt text](/assets/images/1537031357631.png)
