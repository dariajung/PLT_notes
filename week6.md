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
4 : t → Id```
Start with an `e`. We have two rules for what an `e` can become. Let's look at the first rule, `e -> t + e` When we have a choice of a non-terminal to expand, choose the right one. We can get from `e -> e + t`, now we expand `e` to `t`. We currently have `t + t`. Expand the right `t` to a `t`. Expand `t` to `Id`. Now expand the left `t` using rule 3 to `Id * t`. Now we have `Id * t + Id`. Expand `t` to `Id`, which gives us `Id * Id + Id`.
See slides starting from 96 to see colored derivation.
Because we did a right most derivation, we only have to expand left side, and terminals on the right.
######Reverse Rightmost Derivation
Work backwords to re-derive `Id * Id + Id`.
######Shift/Reduce Parsing Using an Oracle
Shift, put onto top of stack. Reduce, take off from top of stack. 
Oracle is very clever. See it how it shifts/reduces on slides.
Later we'll be told how to build this Oracle.
---
October 8, 2014
######Parsing Algorithms
Last class we talked about shift/reduce parsing. What's going on with the oracle? It's just an automaton. 
######Handle Hunting
Right Sentential Form: any step in a rightmost derivation
Handle: in a sentential form, a RHS of a rule that, when rewritten, yields the previous step in a rightmost derivation. But how do we figure out what is and isn't a handle? When is there a handle on the top of the stack? 
Naive: let's write down all right-sentential forms and pattern match against them? This is crazy, but let's try it anyway. Usually you'll end up with an infinite number of things. See slide 122 on syntax.pdf.
When we start expanding the tree, while it goes on infinitely, we start seeing patterns.
A regular deterministic automaton suffices to locate a handle in a right sentential form - Donald Knuth
*Look at automaton on slide 125. 
We could do a left-most derivation to get a left sentential form. 
CFG --> Push down automaton, where do we push? But it just so happens that this grammar can be expressed as a regular expression. 
So how do we get this automaton? The algorithm can be found in the Dragon Book (Compilers).
######Building Initial State of LR(0) Automaton
If Pacman is to the left side of a non-terminal, then Pacman "eats" the expression;  any viable prefix must be at the beginning of a string expanded from e.
In the trees, we always start with e, t or Id. Pacman can chew up an e, t, or Id. If there's an e on your stack, then you're done (so long as there is no more input). If Pacman is about to "starve" or is satiated, then that is an accepting state. Keep doing this until we can't generate any more new states. See slides starting at 130 for reduction diagrams.
Once we've built automaton, what do we do? We need two functions, one of which is the first function.
######First Function
If you can derive a string that starts with terminal t from some sequence of terminals and nonterminals a, then t belongs to first(a).
A terminal can only start with a terminal. 
We're interested in the follow set.
######Follow Function
We're going to use this follow set, if we just finished this viable prefix, something is wrong if the next token is not in the follow set of that viable prefix. Something has gone wrong, there is a syntax error. Can we safely reduce a handle without breaking right most derivation? For each terminal, we look at what can follow it. If a non-terminal appears at the end of a rule, we have to look at the context in which it occurs.

Let's look at: Id + Id Id
Id + Id could be turned into an e. But e Id is a syntax error. There is no way to be able to get two IDs in a row.
What could follow a t is something that could also follow an e. 
Take the automaton and put it into a table.
`$` is a stop symbol.

######Converting LR(0) Automaton to SLR Parsing TableIf terminal, either reduce using some rule, or shift. For non-terminals, tell it what state to go to.
So probably study how to make an SLR table since Professor Edwards keeps hinting it will be on a test. 
***Always reduce the handle on the top of the stack, not something deep in the stack.