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




