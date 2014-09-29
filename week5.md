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

Kleene closure: Prof Edwards says "it kind of looks like a smiling guy with glasses."

The NFA on slide 29 is wrong because it can accept strings like abab, which is not in the language `a*b*`.

Simulating an NFA:

Follow epsilon closures as far as you can go.

Restricted form of NFAs (a DFA):

- No state has a transition on epsilon
- For each state s and symbol a, there is at most one edge labeled a leaving s.

You don't always have to have an edge leaving a state, because that introduces a dead state.