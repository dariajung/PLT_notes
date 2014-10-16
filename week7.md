###Programming Languages and Translators
---
October 13, 2014

#####The Micro C Compiler

Use the Micro C Compiler as a starting point. (Types are missing though)

A very stripped down dialect of C. It has: functions, global variables, most expressions and statements, but only integer vars

```c
/* The GCD algorithm in MicroC */
gcd(a, b) {
while (a != b) {
if (a > b) a = a - b;
else b = b - a;
}
return a;
}
main()
{
print(gcd(2,14));
print(gcd(3,15));
print(gcd(99,121));
}
```

No types mentioned because everything is an integer.

#####Front-End

Tokenize and parse to produce an Abstract Syntax Tree.

MicroC slide 4 to see scanner.mll

How do we get an arbitrary number of nesting? Prof Edwards says don't worry about it unless you really care. 

#####Structure of AST 

- the crispest vision of the structure of your programming)
- As few cases as possible
- Keep special cases to a minimum
- No idea that you have a standard library in your AST
- Think about if your language need statements AND expressions. OCaml only has expressions.
- Program is a list of strings and a list of function declarations

A good test is to have a pretty printer. Can your compiler take take something, you pretty print it, put it back into your compiler, and it is happy? Good test to make sure your AST is good. Pretty printer prints out your program.

#####Parser (Parser builds the AST)
Token order has no precedence, but untyped tokens should be separated from ones with types.

You need to specify the type of the root

#####Function Declarations
Create lists, then reverse to preserve natural order (reversing is a linear operation).

Question asked in class: What is a body? List of statements, and the natural way to build up the list of statements is in reverse order. So we reverse the list when we look at it. 

`%prec`: treat this statement with the precedence of NOELSE. 

Where do variable declarations go? Introduce a variable declarations statement anywhere where the statement can happen. 

How do we solve the elseless if/ dangling if problem? Assign a NOELSE precedence (makes shift reduce go away wee).

#####Expressions
Literals, Identifiers, operators

What is actuals_list? actuals_opt?

`f(x) = x + 1` When you give a function parameter names, they aren't the actual values, they're the "formals" vals. So when you get an actual argument that you pass into the function, they are the "actuals".

Matter of taste what to call these:

- Parameters == Formals
- Arguments == Arguments

#####AST for GCD Example
See slide 10 for the Abstract Syntax Tree. 

#####Interpreter 
Prof Edwards showing this to us, so we don't do this. 

Interpreters are slowwww.

MicroC has global and local variables. Local variables are only in the scope of the function they were declared. 

We need to update the environment, because there might be side-effects.

```
a = 2
print(a = 3)
a = a + 1
a will be 3 + 1 (4)
```

Ew side effects. 

#####Eval: handling variable names
Takes environment and variable name. Checks to see if variable in locals or globals. Goes and gets that if in local, if not in there, goes to global. Otherwise, throws an exception. Think about how to deal with globals and locals. In MicroC we look at local vars before global vars. 

Be consistent in how you check where variable is. 

Consider differences for user and language designer. 

----

October 15, 2014

What is AST vs Parser?

Parser generates the AST. AST is a datatype/elaborate data structure. 

Remember, don't implement an interpreter for your language. 

```
a = 1
a + (a = 3)

What would you like this to return as the language designer? 
```
In MicroC, e1 gets operated first in Binop(e1, op, e2)

What happenes when we call a function? Need to get semantics of function call head on. 

- If there is no function foo, then throw exception (look in func_decls)
- Difference between passing expression to function, or value of expression to function. 

```
a = 2
foo(a = 3, a + 1)
```

Do we want foo(3, 4)? Evaluate left to right.

How do we make the return statement? Let's look at rule for while loop.

Execute statements using a fold left. Whee. Pass in original env as accumulator. 

Function definition, where a, b, and c are formal arguments.

```
foo(a, b, c) {
}

3 + foo(3, 5, 4)

When we call foo, within the scope of the function call, a should be 3, b should be 5, c should be 4.
```

Compilers don't execute like interpreters; rather than doing the computation now, LATER do the computation.

So what does a compiler look like?

#####A Bytecode Compiler
There are many outputs from a compiler. Could be Assembly, C, Bytecode, etc.

Compilation: a syntax-directed translation into a more detailed representation of the program 

Take the AST and run it through a big recursive function. Take the literal 3, and 4, and add them. 

This is a stack based, but not the most efficient to implement in hardware. But an easy target for code generation.

One main big stack. Two registers. Stack pointer points to the first thing not allocated onto the stack. Frame pointer is beneath the stack pointer. Activation record (where all the information is to let you execute a function)

Two things to do when you're working on project: add another level of indirection, and add another new data type

Push/discard from a stack (you should know this!)

Subroutines example:

Going back to `3 + foo(3, 5, 4)`

```
Push Lit 3
Push Lit 4
Push Lit 5
Push Lit 3

Jump to subroutine where foo is

And then call Add
```

Look at slides for Bytecode for GCD (slide 26)


