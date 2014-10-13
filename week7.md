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