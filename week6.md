#####Programming Languages and Translators
----
October 6, 2014

######Modeling Sentences
---
- Richer sentences are harder.
- Automata have poor memories (you need to have separate states to "remember", which is kind of annoying)
- Automata do not remember where they've been. 

A way to think about this is to have terminals, which lead to another state, though this doesn't fully address the issue. Kind of like go-tos.

This is where context free grammars come in handy. A bit like subroutines.

See slide 72 of `syntax.pdf` for a Context Free Grammar for a simplified C. Use CFG to get nesting, structure, etc.

######OCamlyacc
---
`yacc`: Yet Another Compiler Compiler

- The name is kind of silly. It's a parser generator.
- This is the path of least resistence. 

######Parsing

Given a string of tokens, is it represented by the CFG for whatever language you're asking about? This is a decision problem. Is it syntactically correct? If it is, *why* is it correct?

Build an Abstranct Syntax Tree. 

Challenge with creating CFG is avoiding ambiguity. A computer has no common sense. 

```
One morning I shot an elephant in my pajamas.
How he got in my pajamas I don’t know. —Groucho Marx
```
Did I shoot an elephant (with a gun while wearing) my pajamas? Or did I shoot an elephant who somehow is wearing my pajamas? English grammar is ambiguous!!!

######The Dangling Else Problem

Who owns the *else*?

`if (a) if (b) c(); else d();`

Who gets the `else`? The C programming language manual states: "As usual the “else” is resolved by connecting an else with the last encountered elseless if." So we want the first tree on slide 78 of syntax.pdf.

In Ocaml yacc we want to write something like 

```ocaml
stmt : IF expr THEN stmt
| IF expr THEN stmt ELSE stmt
```

Some languages resolve this problem by insisting on nesting everything. 

Let's get fancier.

######Another Solution to the Dangling Else Problem

Break into two types of statements, one that has danging "then", and one that doesn't.

```ocaml
stmt : dstmt
| cstmt
dstmt : IF expr THEN stmt
| IF expr THEN cstmt ELSE dstmt
cstmt : IF expr THEN cstmt ELSE cstmt
| other statements...```

######Ambiguous Arithmetic
Ambiguity also seen in arithmetic expressions.

Consider: `3 - 4 * 2 + 5` and the grammar `e → e + e | e − e | e ∗ e | e / e | N`

The grammar corresponds to the first tree on slide 82 on syntax.pdf.

######Operator Precedence and Associativity

Defines how "sticky" an operator is. 

`*` has a higher precedences than `+`.

`(1 * 2) + (3 * 4)`

######Associativity
Whether to evaluate left-to-right or right-to-left. 
Most operators are left-associative, but subtraction is a bit different. 

`1 - 2 - 3 - 4`

We want the arithmetic expression to have one answer. We don't want to question how the compiler might interpret an expression.

Certain operators need precedence and associativity.

Oh nooooo, shift/reduce conflicts :( :( :( The grammar is broken, the grammar is ambiguous. Yacc is telling you to fix it. 

######Assigning Precedence Levels

```ocaml
expr : expr PLUS expr
| expr MINUS expr
| term
term : term TIMES term
| term DIVIDE term
| atom
atom : NUMBER
```

Something like 1 + 2 + 3. Which way do we parse it? Do we evaluate 1 + 2 first? 

Make it so that you can only build up sequences from the left or from the right. 

Update above parser to:

```ocaml
expr : expr PLUS term
| expr MINUS term
| term
term : term TIMES atom
| term DIVIDE atom
| atom
atom : NUMBER
```

Now expr has precedence over term.

You want left leaning trees.

######OCamlyacc Specifications

```ocaml
%{
(* Header: verbatim OCaml; optional *)
%}
/* Declarations: tokens, tokens with types, precedence, etc. See more on slide 91 titled Declarations in syntax.pdf */
%%
/* Rules: context-free rules */
%%
(* Trailer: verbatim OCaml; optional *)
```

You need to tell yacc what the tokens are in an `.mly` file. 

Fix ambiguity by restructing grammar, or by explictly listing precedence rules. ie:

```ocaml
%left PLUS MINUS
```

.mly file is an Ocaml yacc file. 

Can you get the grammar through yacc without shift/reduce errors? Do this for your project. A parser generator is a useful tool since we need to do this for our languages anyway.

######Parsing Algorithms

Where are these shift/reduce errors coming from? 

Subclasses of Context Free Grammars have O(n) parsing algorithms. Yay so fast. 

######Rightmost Derivation of Id * Id * Id

A parser effectively needs to construct a proof. 

`e` and `t` are non-terminals. `+`, `*`, and `Id` are terminals. 

```ocaml
1 : e → t + e
2 : e → t
3 : t → Id ∗ t
4 : t → Id```


