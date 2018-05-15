---
layout: post
title: "Lexical Analysis "
description: "Lexical Analysis"
categories: [Compiler]
tags: [Compiler]
redirect_from:
  - /2018/05/10/
---

## Lexical Analysis
**goal**: identify each lexeme and return information on token
### 1 Lexical Analyzer Role
(1) read input characters, group them into lexemes, and produce output a sequence of tokens

(2) When the lexical analyzer discovers a lexeme constituting an identifier, it needs to enter that lexeme into the symbol table. 

![image](/assets/images/1525905998501.png)

The figure suggests, parser will call lexical analyzer, and lexical analyzer read characters until find next token. 
lexical analyzer will also do some additional jobs:

(1) strip whitespace

(2) record line number for error message
#### 3.1.1 Basic Concepts
(1) A **token** is a pair consisting of a token name and an optional attribute value. The token name is an abstract symbol representing a kind of lexical unit. 

(2) A **pattern** is a description of the form that the lexemes of a token may take. 
the sequence of character which can form keywords

(3) A **lexeme** is a sequence of characters in the source program that matches the pattern for a token and is identified by the lexical analyzer as an instance of the token. 

In most programming languages, the following classes cover most or all of the tokens:

(1) One token for each keyword. The pattern for a keyword is the same as the keyword itself.

(2) Tokens for the operators

(3) One token representing all identifiers.

(4) One or more tokens representing constants

(5) To kens for each punctuation symbol, such as left and right parentheses, comma, and semicolon.

Sometimes, token can have different matches, for example, 0 and 1 both match for **number**. So we not only need token names, we also need token attributes. The token name influences parsing decisions, while the attribute value influences translation of tokens after the parse. 
The appropriate attribute value for an identifier is a pointer to the symbol table entry for that identifier. 

**Lexical Errors** suppose a situation arises in which the lexical analyzer is unable to proceed because none of the patterns for tokens matches any prefix of the remaining input. The simplest recovery strategy is "panic mode" recovery. We delete successive characters from the remaining input, until the lexical analyzer can find a well-formed token at the beginning of what input is left. This recovery technique may confuse the parser, but in an interactive computing environment it may be quite adequate.

### 2 Input Buffer
In Dragon book, we can see the example of Fortran language, which justifies the need to look ahead to help tokenize strings. 
1. The goal is to partition the string. This is implemented by reading left-to-right, recognizing one token at a time. 
2. "Lookahead" may be required to decide where one token ends and the next token begins. 

There are many situations where we need to look at least one additional character ahead. 
![image](/assets/images/1525925908025.png)

We can use a buffer pair to resolve the problem. Each buffer is of the same size N, which is usually the size of a disk block. Using one system read command we can read N characters into a buffer, rather than using one system call per character. 

In the buffer, two pointers are maintained. 

(1) Pointer lexemeBegin, marks the beginning of the current lexeme, whose extent we are attempting to determine.

(2) Pointer forward scans ahead until a pattern match is found; the exact strategy whereby this determination is made will be covered in the balance of this chapter.

![image](/assets/images/1525926047058.png)
We can add **sentinels** at the end of each buffer. 
```
switch (*forward++) {
	case eof: 
		if (forward is at end of first buffer) {
			reload second buffer;
			forward = beginning of second buffer;
		}
		else if (forward is at the end of second buffer) {
			reload first buffer;
			forward = beginning of first buffer;
		}
		else 
			terminate lexical analysis
		break;
	case for other characters;
}
```
### 3 Token Specification
An **alphabet** is any finite set of symbols. 
A **string** over an alphabet is a finite sequence o symbols drawn from the alphabet. 
A **language** is any countable set of strings over some fixed alphabet. 

**operations on language**
![image](/assets/images/1525926320045.png)
#### 3.1 Regular Expression
We are able to describe identifiers by giving names to sets of letters and digits and using the language operators union, concatenation, and closure. This process is so useful that a notation called **regular expression** has come into common use for describing all the language that can be built from these operators applied to the symbols of some alphabet.  
Here are the rules that define the regular expressions over some alphabet $\Sigma$ and the languages that those expressions denote. 
There are two rules that form the basis:

(1) $\epsilon$ is a regular expression, and $L(\epsilon)$ is $\{\epsilon\}$, that is, the language whose sole member is empty string. 

(2) If $a$ is a symbol in $\Sigma$, then $a$ is a regular expression, and $L(a) = \{a\}$, that is, the language with one string, of length one, with $a$ in its one position. 

![image](/assets/images/1525926883438.png)
![image](/assets/images/1525926900662.png)
#### 3.2 Extensions of Regular Expressions
(1) One or more instances. `(r)+`

(2) Zero or one instance. `r?`

(3) Character classes `[abc]`

### 4 Recognition of Tokens
Here is our example:
```
stmt -> if expr then stmt
	 | if expr then stmt else stmt
	 |  $\epsilon$
expr -> term relop term
	 | term
term -> id
	 | number 
ws -> (blank | tab | newline)+
```
![image](/assets/images/1525928409876.png)
### 5 Finite Automata
The heart of transition is the formalism known as **finite automata**. 
Finite automata come in two flavors:

(1) **Nondeterministic finite automata**(NFA) have no restrictions on the labels of their edges. A symbol can label several edges out of the same state, and $\epsilon$, the empty string, is a possible label. 

(2) **Deterministic finite automata** (DFA) have, for each state, and for each symbol of its input alphabet exactly one edge with that symbol leaving the state. 

Both deterministic and nondeterministic finite automata are capable of recognizing the same language. In fact these language are exactly the same language, called the **regular language**, that regular expressions can describe. 
#### 5.1 NFA
NFA consists of :
(1) A finite set of states $S$

(2) A set of input symbols $\Sigma$, the input alphabet. We assume that $\epsilon$, which stands for the empty string, is never a member of $\Sigma$

(3) A transition function that gives, for each state, and for each symbol in $\Sigma \cup\{\epsilon\}$ a set of next states. 

(4) A state $s_0$ from $S$ that is distinguished as the start state(or initial state)

(5) A set of states $F$, a subset of $S$, that is distinguished as the accepting states(or final states)

An NFA **accepts** input string $x$ iff there is some path in the transition graph from the start state to one of the **accepting states**, such that the symbols along the path spell out $x$. 
#### 5.2 DFA
A deterministic finite automata is a special case of an NFA where 

(1) There are no moves on input $\epsilon$

(2) For each state $s$ and input symbol $a$, there is exactly one edge out of $s$ labeled $a$. 

Every regular expression and every NFA can be converted to a DFA accepting the same language, because it is the DFA that we really implement or simulate when building lexical analyzers. 
```
# Simulate a DFA
s = s_0;
c = nextChar();
while (c != eof) {
	s = move(s, c);
	c = nextChar();
}
if (s in Final_state) return "yes";
else return "no";
```
### 6 Regular Expression -> Automata
#### 6.1 NFA->DFA
The general idea behind the subset construction is that each state of the constructed DFA corresponds to a set of NFA states. After reading input $a_1a_2...a_n$, the DFA is in that state which corresponds to the set of states that NFA can reach, from its start state, following paths labeled $a_1a_2...a_n$. 
![image](/assets/images/1525974027615.png)

Before reading the first input symbol, N can be in any of the states of $\epsilon-closure(s_0)$. For induction, suppose N can be in set of states $T$ after reading input string $x$. If it next reads input $a$, then $N$ can immediately go to any of the states in $move(T,a)$. However, after reading $a$, it may also make $\epsilon$ transitions. Thus $N$ could be in any state of $\epsilon-closure(move(T,a))$ after reading input $xa$. 
How to construct the set of $D$'s states, $Dstates$, and its transition function $Dtran$. 
Initially, $\epsilon-closure(s_0)$ is the only state in $Dstates$ and it is unmarked. 

```
# construct Dstates and Dtran
while (there is an unmarked state T in Dstates) {
	mark T;
	for (each input symbol a) {
		U  = epsilon-closure(move(T, a));
		if (U is not in Dstates)
			add U as an unmarked state to Dstates;
		Dtran[T,a] = U;
	}
}
```
```
# compute epsilon closure for T
push all states of T onto stack;
initialize epsilon-closure(T) to T;
while (stack is not empty) {
	pop t off the stack;
	for (each state u with an edge from t to u labeled epsilon) {
		if (u is not in epsilon-closure(T)) {
			add u to epsilon-closure(T);
			push u onto stack;
		}
	}
}
```
![image](/assets/images/1525975686530.png)

For line 1 and line 4 we can implement a function `addState(s)`. This function pushes state $s$ onto `newStates`, sets `alreadyOn[s]` to `TRUE`, and calls itself recursively on the states in `move[s,epsilon]` in order to further the computation of $\epsilon-closure(s)$
```
addState(s) {
	push s onto newStates;
	alreadyOn[s] = True;
	for (t on move[s, epsilon])
		if (!alreadyOn(t))
			addState(t);
}
```
For line 4
```
for (s on oldStates) {
	for (t on move[s,c]) 
		if (!alreadyOn[t])
			addState(t);
	pop s from oldStates;
}
for (s on newStates) {
	pop s from newStates;
	push s onto oldStates;
	alreadOn[s] = FALSE;
}
```
#### 6.2 Regular Expression -> NFA
![image](/assets/images/1525976909215.png)
![image](/assets/images/1525977147532.png)
![image](/assets/images/1525977159550.png)
![image](/assets/images/1525977172727.png)

NFA properties:
(1) $N(r)$ has at most twice as many states as there are operators and operands in $r$. This bound follows the fact that each step of the algorithm creates at most two new states

(2) $N(r)$ has one start state and one accepting state. The accepting state has no outgoing transitions, and the start has no incoming transitions. 

(3) Each state of $N(r)$ other than the accepting state has either one outgoing transition on a symbol in $\Sigma$ or two outgoing transitions, both on $\epsilon$. 

### 7. Design for a Lexical Generator
![image](/assets/images/1525981672518.png)

The program that serves as the lexical analyzer includes a fixed program that simulates an automaton. We need a single automation that will recognize lexemes matching any of the patterns in the program, so we combine  all the NFA's into one by introducing a new start state with $\epsilon$-transitions to each of the start states of the NFA's $N_i$. 
