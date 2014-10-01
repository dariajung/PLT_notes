####PLT Week 5
---
September 29, 2014

The next step of the group project is the language reference manual. Check in with your TA once a week to make sure you're on the right track for your project. 

How do you represent one of many things? We want our compiler to be able to handle many different programs. How do we describe certain things to it? 

- One way is continuously varying values; it's not a digital thing per se. The density is very high, but there is a lot of noise / little variations that shows up as data. 
- ENIAC: take wires and put them in certain places
- Different symbol per thing you want to represent. Works well when only a few things. 

What is the solution? Use a discrete combinatorial system (use combinations of a small number of things to represent MANY different things)

Every human writing system falls into this idea.

How do you describe only certain combinations? 
Compilers should only accept correct programs; how should a compiler check that its input is correct?

- One thing to do is list all of them. Which is super annoying.

See Professor Edwards' slides `syntax.pdf` for examples of different ways of how to generate combinations.

#####Lexical Analysis
---
Take a stream of characters and translate to a stream of tokens. 

Goal: simplify the job of the parser and reject some wrong programs. 

######Describing Tokens:

Alphabet: A finite set of symbols.
- ie: {0, 1}, {A, B, C, ..., Z}

String: A finite sequence of symbols from an alphabet. Epsilon stands for the empty string. 

######Language: A set of strings over an alphabet. 

**The alphabet is finite, but the language itself might be infinite.**

######Operations on Languages:

Concatenation: Strings from one followed by the other.
ie: wo + man = woman. 

LM = {man, men, woman, women} 

Union: All strings from each language.

L U M = {epsilon, wo, man, men}

Kleene Closure: Zero or more concatenations. This is an infinite language, although if we pick out a particular string in this language, it is finite.

#####Kleene

Kleene is pronounced "clay-nee". 

#####Regular Expressions over an Alphabet 

A standard way to express languages for tokens.

See slides starting on pg 23 of Professor Edwards' slides.

A regular expression example gone over in class:

- `a|a*b = a or a* followed by b` <=> `(a) | (a*)(b)`

#####Implementing a Scanner

Describing tokens with regexes to implement a scanner.

Regular Expression (Rules) ---> Nondeterministic Finite Automata ---> Subset Construction ---> Deterministic Finite Automata ---> Tables

For project, come up with tokens that describe your project. Type up regexes in textfile and pass it to OCaml Lex. 

Recall CS Theory.

######NFA needs five things (see slide 26):

1. Set of States
2. Set of input symbols/the alphabet
3. Transition function
4. Start state
5. Set of accepting states

An NFA accepts an input string `x` iff there is a path from the start state to an accepting stat that "spells out" `x`.

Translating REs into NFAs (Thompson's algorithm) slide 28.

Epsilon transitions? Input on no input at all? This means you're in two different states at once (if there are two epsilon transitions). 

Kleene closure: Prof Edwards says "it kind of looks like a smiling guy with glasses." The mouth is the empty case, and the bridge of the glasses is the repeat case.

The NFA on slide 29 is wrong because it can accept strings like abab, which is not in the language `a*b*`.

Simulating an NFA:

Follow epsilon closures as far as you can go.

Restricted form of NFAs (a DFA):

- No state has a transition on epsilon
- For each state s and symbol a, there is at most one edge labeled a leaving s.

You don't always have to have an edge leaving a state, because that introduces a dead state.

#####October 1, 2014

Last class, we talked about lexical analysis. Taking a stream of characters and turning them into a stream of tokens. We want to formalize patterns more precisely and write code that takes these strings of characters and turns them into tokens.

We'll talk about subset construction today.

A DFA is a restricted NFA where you can only be in one state at one time.

Rather than trying to emulate possibilities at a time, we can try to simulate them at the same time.

Subset construction for `(a|b)*abb`. We only have two input possibilities, `a` or `b`. We want to construct a path through the NFA, but we don't know exactly which path we are going to follow. Rather than being absolutely sure where we are, we can just hand-wavey-y talk about the existence of a path (rather than knowing exactly what the path is itself).

See slides for subset construction diagrams.

We can construct a DFA by just creating a loop on a state you've already seen before.

Question asked in class: Is there a way to create the worst possible DFA? Yes, by unrolling the DFA as many times as we want. Not sure why you'd want to do that. Worst case, it's exponential.

DFA minimization is usually polynomial time.

Look in the Dragon book for a different way of laying out the subset construction algorithm.

How big does the NFA get in relation to the number of regular expressions? Linear. But the DFAs have a potential to grow.

We will see the subset algorithm popping up elsewhere in the future. 

**PRACTICALLY:**

Write a `.mll` file, run it through the ocamllex program. It will produce the `.ml` file that essentially builds you the parser/tokenizer.

```ocaml
{ open Parser }
rule token =
parse [’ ’ ’\t’ ’\r’ ’\n’] { token lexbuf }
| ’+’ { PLUS }
| ’-’ { MINUS }
| ’*’ { TIMES }
| ’/’ { DIVIDE }
| [’0’-’9’]+ as lit { LITERAL(int_of_string lit) }
| eof { EOF }
```

See slide 58 for possible patterns that can be used in OCamllex. 

Your rules should look something like this for your project:

```ocaml
{ type token = PLUS | IF | ID of string | NUM of int }
let letter = [’a’-’z’ ’A’-’Z’]
let digit = [’0’-’9’]
rule token =
parse [’ ’ ’\n’ ’\t’] { token lexbuf } (* Ignore whitespace *)
| ’+’ { PLUS } (* A symbol *)
| "if" { IF } (* A keyword *)
(* Identifiers *)
| letter (letter | digit | ’_’)* as id { ID(id) }
(* Numeric literals *)
| digit+ as lit { NUM(int_of_string lit) }
| "/*" { comment lexbuf } (* C-style comments *)
and comment =
parse "*/" { token lexbuf } (* Return to normal scanning *)
| _ { comment lexbuf } (* Ignore other characters *)
```

#####Free-Format Languages
Program text is a series of tokens possibly separated by whitespace and comments, which are both ignored. Python and OCaml are not free-format languages because indentation matters. Free-format languages include C, Java.

To this end, they need:

- Keywords (if, while)
- punctuation (, ( +)
- identifiers (foo bar)
- numbers (10 -3.14)
- strings ("A string")

Fortran is not free format, which is probably good because FORTRAN code is punched onto paper and it would be bad to go back and have to re-punch holes into a card again.

#####Syntax and Language Design
The semantics are most important. Syntax is kind of like a religion. Aesthetics matter to people. Verbosity matters. Too small can be problematic though. 

Some syntax is error prone, ie: angle brackets in C++.

What does something like "ifa" return? It should return one token
"if+" should return two tokens.

Keywords look like identifiers, but are more specific. 