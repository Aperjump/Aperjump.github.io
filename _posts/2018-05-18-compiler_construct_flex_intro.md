---
layout: post
title: "Constructing Compiler: flex intro "
description: "Constructing Compiler: flex intro"
categories: [Compiler]
tags: [Compiler]
redirect_from:
  - /2018/05/18/
---
### Flex intro
**flex** is a tool for generating **scanners**. This article is my notes for using **flex**, and most of the material can be found in [website](http://westes.github.io/flex/manual/)
```
/* flex syntax */
definitions
%%
rules
%%
user code
```
When the scanner is run, it analyzes its input looking for strings which match any of its patterns. If it finds more than one match, take the longest match token. 
One the match is determined, the text corresponding to the match(**token**) is made available in the global character pointer **yytext**, and its length in the global integer **yyleng**. The **action** corresponding to the matched pattern is then executed. 
If no match is found, then the default rule is: **the next character in the input is considered matched and copied to the standard ouput.**
**yytext** can be defined as a pointer or array
use `%pointer` or `%array`

### 1. actions
```
 %%
[ \t]+        putchar( ' ' );
[ \t]+$       /* ignore this token */
```
The action consisting solely of a vertical bar `('|')` means "same as the action for the next rule". 
Actions can include arbitrary C code, including `return` statements to return a value to whatever routine called `yylex()`. Each time `yylex()` is called it continues processing tokens from where it last left off until it either reaches the end of the file or executes a return. 
More detailed `action` description can be found [here](http://westes.github.io/flex/manual/Actions.html#Actions)

### 2. generated scanner
output file: `lex.yy.c`(includes scanning routine `yylex()`, symbol table, and macros)
whenever `yylex()` is called, it scans tokens from the global input file `yyin`(which defaults to `stdin`). It continues until it either reaches an end-of-file or one of its actions executes a `return` statement. 
If the scanner reaches an end-of-file, subsequent calls are undefined unless either `yyin` is pointed at a new input file, or `yystart()` is called. `yystart()` takes one argument, a `FILE *` pointer, and initializes `yyin` for scanning from that file. 
By default, the scanner uses block-reads rather than simple `getc()` calls to read characters from `yyin`. This is controlled by defining `YY_INPUT` macro. `YY_INPUT(buf, result, max_size)`its action is to place up to `max_size` characters in the `buf` and return in the integer variable `result` 
```
%{
 #define YY_INPUT(buf,result,max_size) \
     { \
     int c = getchar(); \
     result = (c == EOF) ? YY_NULL : (buf[0] = c, 1); \
     }
 %}
```
### 3. start conditions
`flex` provides a mechanism for conditionally activating rules. Any rule whose pattern is prefixed with `<sc>` will only be active when the scanner is in the start condition named `sc`.
```
<STRING>[^"]* {
}
```
will be active only when the scanner is in the `STRING` start condition. 
Start conditions are declared in the definitions section of the input using lines beginning with either `%s`(inclusive) or `%x`(exclusive). 
A start condition is activated using the `BEGIN` action.  If the start condition is inclusive, then rules with no start conditions at all will also be active. If it is exclusive, then only rules qualified with the start condition will be active.
```
%{
#include <math.h>
%}
%s expect

%%
expect-floats        BEGIN(expect);

<expect>[0-9]+.[0-9]+      {
            printf( "found a float, = %f\n",
                    atof( yytext ) );
            }
<expect>\n           {
            /* that's the end of the line, so
             * we need another "expect-number"
             * before we'll recognize any more
             * numbers
             */
            BEGIN(INITIAL);
            }

[0-9]+      {
            printf( "found an integer, = %d\n",
                    atoi( yytext ) );
            }

"."         printf( "found a dot\n" );
```