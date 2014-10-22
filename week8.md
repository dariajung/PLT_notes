####Programming Languages and Translators
---

October 20, 2014

What is the AST going to be like for my language?

If you're doing recursion, it makes a lot of sense to use a stack. 

######expr: Code for literals and identifiers
Look for cases 

- Literal
- Id
- Assign
- Noexpr
- Binop
- Call: calling a function, push the actual arguments onto the stack in reverse order

######Statements
For blocks, concatenate the list of statements together

```
If (predicate, true, false)
```

Glue these together and test the expression; if statement returns false, jump over true block, and execute false block.

Take a for loop and transform it into a little block with a while loop; slows down the compiler a wee bit, but it's worth it to write a separate rule to generate code for the for loop which would look like a while loop with extra junk.

######Bytecode Interpreter
`exec` ends with a tail recursive call

Expect to have about 2000 lines of code for project. Don't repeat yourself.

######Notion of Type
A type is a restriction on how you interpret something. 
Two uses: Safety, Optimization

Different types of types

- Basic: Machine words, floating-point-numbers, addresses/pointers
- Aggregate Arrays, structures, classes
- Function Function pointers, lambdas

These are the ones where you can write single assembly language instructions.

ARM processor (32 bit machines)

- Bytes: 8 bytes
- Half word: 16 bytes
- Word: 32 bytes

C's Declarations and Declarators

`static unsigned int(*f[10])(int, char*);`

Fake object-oriented programming (C++ --> C)

union holds only one of its fields at any time. 

```c
struct f {
int x, y;
} foo = { 0, 1 };
struct b {
int x, y;
} bar;
bar = foo;```Is this legal in C? These two are structurally equivalent. C compares types by name, not by structure. Error: Incompatible types 
---######October 22, 2014

If you're going to have operators, make them just postfix, or just prefix.

C's declarators, you have to give them a name. Most of the this is fine. So you have a type expression.

You can't pass types to a function, you can only pass parameters. Something like sizeof takes type expression: `sizeof(float [10])` and returns the number of bytes. 

`char *` is a type expression.

Come up with a syntax for types, and values. 

######Semantic Checking: Static vs Dynamic typing

Build an AST, pass it through static semantic analysis to make sure AST has right structure. 

- Lexical analysis: Make sure tokens are valid
- Syntactic analysis: Make sure tokens appear in correct order
- Semantic analysis: makes sure program is consistent

AST to Semantically checked AST. 

Write some sort of checking function: pass an environment into it, pass it an AST node, and a type.

If we get a name of a variable, then we need to check what its type is. 

######Static Scoping 
A name begins life where it is declared and ends at the end of its block.

Static scoping means you can derive what's in what scope just by looking at the structure of the code. 

######Basic Static Scope in OCaml
A name is bound after the “in” clause of a “let.” If the name is re-bound, the binding takes effect after the “in.”

If you're having user defined functions in your language, will you let your user nest functions?

#####A Static Semantic Analyzer

Check: ast -> sast (semantic Abstract Syntax Tree)

In an expression, you want every subnode to have its own type.

`type` is a keyword in OCaml

Symbol Table: It's a map from names to information about the names, typically the types, or where the things get laid out in memory. The symbol table should tell you all you need about that name. 

How to find something in a symbol table? Or update it? Maybe you don't need a map, just use a simple linear search (list or array). Add a child symbol table when you enter a new scope. When you leave scope, throw away the child. 

What do you do when you come across a block? The parent is the existing scope, so you create a new scope when the parent is the scope you currently have. The new local thing is empty. 

Build a static semantic analyzer for your compiler. Expressions (literals and identifiers) --> when you come across name, find it; Binary operators; when calling a function, make sure it's already defined

Figure out what has to be right about the names. 