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
---


