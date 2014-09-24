####Programming Languages and Translators
---
Week 4, September 22, 2014

Pattern Matching with `when` and `as`

The when keyword lets you add a guard expression:

```ocaml
let tall = function
	| (h, s) when h > 180 -> s ^ " is tall"
	| (_, s) -> s ^ " is short";;
```

```ocaml
match ((3,9), 4) with
	(_ as xx, 4) -> xx
```

#####Type Declarations 

Algebraic Data types

`type name = typedef`

#####Records
If you have two or three fields, you can use tuples, but if you can't remember, you can declare types this way:

```ocaml
# type base = { x : int; y : int; name : string };;
type base = { x : int; y : int; name : string; }
# let b0 = { x = 0; y = 0; name = "home" };;
val b0 : base = {x = 0; y = 0; name = "home"}
# let b1 = { b0 with x = 90; name = "first" };;
val b1 : base = {x = 90; y = 0; name = "first"}
# let b2 = { b1 with y = 90; name = "second" };;
val b2 : base = {x = 90; y = 90; name = "second"}
# b0.name;;
- : string = "home"
```

Make names of fields globally unique.

#####Algebraic Types/Tagged Unions/Sum-Product Types

You can define enumerated types; objects that can be one of a set of types.

Boolean is also an enumerated type:

```ocaml
type bool = True | False
```

`Winter` on its own is a type constructor.

```ocaml
# type seasons = Winter | Spring | Summer | Fall;;
type seasons = Winter | Spring | Summer | Fall
# let weather = function
Winter -> "Too Cold"
| Spring -> "Too Wet"
| Summer -> "Too Hot"
| Fall -> "Too Short";;
val weather : seasons -> string = <fun>
# weather Spring;;
- : string = "Too Wet"
# let year = [Winter; Spring; Summer; Fall] in
List.map weather year;;
- : string list = ["Too Cold"; "Too Wet"; "Too Hot"; "Too Short"]
```

Syntax trees:

```ocaml
# type expr =
Lit of int
| Plus of expr * expr
| Minus of expr * expr
| Times of expr * expr;;
type expr =
Lit of int
| Plus of expr * expr
| Minus of expr * expr
| Times of expr * expr
# let rec eval = function
Lit(x) -> x
| Plus(e1, e2) -> (eval e1) + (eval e2)
| Minus(e1, e2) -> (eval e1) - (eval e2)
| Times(e1, e2) -> (eval e1) * (eval e2);;
val eval : expr -> int = <fun>
# eval (Lit(42));;
- : int = 42
# eval (Plus(Lit(17), Lit(25)));;
- : int = 42
# eval (Plus(Times(Lit(3), Lit(2)), Lit(1)));;
- : int = 7
```
Use pattern matching to create a function `eval`.

How do we change `eval` if we want to return integer or float? When in doubt, create another type (call it `result`) that can be either an integer or a float. `eval` can then spit out a `result` data type.

```ocaml
type result = 	MyInt of int
		| MyFloat of float
```

```ocaml
let rec eval = function 
	Lit(x) -> MyInt x
	.
	.
	.
val eval : expr -> result = <fun>
```

All type constructors/tag name must be upper-case:

```ocaml
# let bad1 = left | right;;
Syntax error
```

Tag names must be globally unique.

Compiler warns about missing cases; if you add new type constructor to `Lit`, then functions that don't cover that new case will return an error.

#####The Option Type: A Safe Null Pointer
`None` is like a null pointer; `Some` is a non-null pointer.

```
`a is a polymorphic type.
```

An example of `Option` in action:

```ocaml
# let rec sum = function
[] -> 0 (* base case *)
| None::tl -> sum tl (* handle the "null pointer" case *)
| Some(x)::tl -> x + sum tl;; (* normal case *)
val sum : int option list -> int = <fun>
# sum [None; Some(5); None; Some(37)];;
- : int = 42
```

Taking a list of `option int` values, and performing sum on it.

See slide 50 of Professor Edwards' OCaml notes to see pros and cons of Algebraic Types vs. Classes and Enums.

An algebraic type is best when the set of types rarely change but you often want to add additional functions. Classes are good in exactly the opposite case.

#####Exceptions
```ocaml
# 5 / 0;;
Exception: Division_by_zero.
# try
5 / 0
with Division_by_zero -> 42;;
- : int = 42
# exception My_exception;;
exception My_exception
# try
if true then
raise My_exception
else 0
with My_exception -> 42;;
- : int = 42
```

You can declare your own exceptions. You can raise them, and you can attach data types to them as well. Exceptions are kind of like algebraic data types. It's nice occasionally if you want a built in function to raise an exception. 

#####Maps
In the OCaml standard library. It's not quite as fast asymptotically as a hash table (logn access time as opposed to constant time). Map.Make isn't really a function (a functor). This is a way to generate a new type (super meta???), no it actually returns an entire MODULE. 

```ocaml
# module StringMap = Map.Make(String);;
module StringMap :
sig
type key = String.t
type ’a t = ’a Map.Make(String).t
val empty : ’a t
val is_empty : ’a t -> bool
val add : key -> ’a -> ’a t -> ’a t
val find : key -> ’a t -> ’a
val remove : key -> ’a t -> ’a t
val mem : key -> ’a t -> bool
val iter : (key -> ’a -> unit) -> ’a t -> unit
val map : (’a -> ’b) -> ’a t -> ’b t
val mapi : (key -> ’a -> ’b) -> ’a t -> ’b t
val fold : (key -> ’a -> ’b -> ’b) -> ’a t -> ’b -> ’b
val compare : (’a -> ’a -> int) -> ’a t -> ’a t -> int
val equal : (’a -> ’a -> bool) -> ’a t -> ’a t -> bool
end
```

How do we use Map? You can't change Maps, but you can take the old Map, make a copy of it, and add stuff to it and make a new Map. Add things to Maps, check for membership, find values via keys.

```ocaml
# let mymap = StringMap.empty;; (* Create empty map *)
val mymap : ’a StringMap.t = <abstr>
# let mymap = StringMap.add "Douglas" 42 mymap;; (* Create new Map, coincidentally called mymap. Add pair *)
val mymap : int StringMap.t = <abstr>
# StringMap.mem "foo" mymap;; (* Is "foo" there? *)
- : bool = false
# StringMap.mem "Douglas" mymap;; (* Is "Douglas" there? *)
- : bool = true
# StringMap.find "Douglas" mymap;; (* Get value *)
- : int = 42
# let mymap = StringMap.add "Adams" 17 mymap;; (* Again, create a new Map. *)
val mymap : int StringMap.t = <abstr>
# StringMap.find "Adams" mymap;;
- : int = 17
# StringMap.find "Douglas" mymap;;
- : int = 42
# StringMap.find "Slarti" mymap;;
Exception: Not_found.
```

Maps are fully functional. Maps need a totally ordered key type. If you really want to make your own Map, you could pass in a struct:

```ocaml
module StringMap = Map.Make(struct
type t = string
let compare x y = Pervasives.compare x y
end)
```

When a Map is empty, it is willing to take any type of KV pair. But once you put in a value, then the Map expects certain types for Keys and Values.

You can do imperative programming in OCaml, but that's not really the point of using OCaml in this class. 
Every once in a while, you might want to print something. It becomes very useful to have functions that have side effects. 
Print debugging is great. :thumbsup:

`print_endline` and `print_string` (both of these take strings, print the strings, and then return the Unit). 

OCaml has Arrays, which are mutable. You can modify an Array in place. 

```ocaml
# let a = [| 42; 17; 19 |];; (* Array literal *)
val a : int array = [|42; 17; 19|]
# let aa = Array.make 5 0;; (* Fill a new array *)
val aa : int array = [|0; 0; 0; 0; 0|]
# a.(0);; (* Random access *)
```

```ocaml
# a.(2) <- 20;; (* Arrays are mutable! Stuff 20 "violently" into where 2 was. *)
- : unit = ()
# a;;
- : int array = [|42; 17; 20|]
```

You can convert between list and Array:

```ocaml
# let l = [24; 32; 17];;
val l : int list = [24; 32; 17]
# let b = Array.of_list l;; (* Array from a list *)
val b : int array = [|24; 32; 17|]
# let c = Array.append a b;; (* Concatenation *)
val c : int array = [|42; 17; 20; 24; 32; 17|]
```

There is a way to forcibly glue stuff to an Array and change its dimensions. 

See slide 61 for Arrays vs. Lists for tradeoffs.

There are Hash Tables. They are also mutable objects like Arrays (Professor Edwards calls them "evil"). 

```ocaml
# module StringHash = Hashtbl.Make(struct
type t = string (* type of keys *)
let equal x y = x = y (* use structural comparison *)
let hash = Hashtbl.hash (* generic hash function *)
end);;
module StringHash :
sig
type key = string
type ’a t
val create : int -> ’a t
val clear : ’a t -> unit
val copy : ’a t -> ’a t
val add : ’a t -> key -> ’a -> unit
val remove : ’a t -> key -> unit
val find : ’a t -> key -> ’a
val find_all : ’a t -> key -> ’a list
val replace : ’a t -> key -> ’a -> unit
val mem : ’a t -> key -> bool
val iter : (key -> ’a -> unit) -> ’a t -> unit
val fold : (key -> ’a -> ’b -> ’b) -> ’a t -> ’b -> ’b
val length : ’a t -> int
end
```

Seed Hash Tables with a reasonable amount of memory. Use HashTables instead of Map if you want mutability or you want something faster. 

Quick discussion of Modules in OCaml:

When you write something using the `<name>.ml` extension, then it automatically turns everything in that file into part of <name> module. So `foo.ml` makes everything within part of the `Foo` module.

If name of the module is long or complicated, you can import qualified:
Import `Foo` as `F`.

```ocaml
module F = Foo;;
```

Use `open` to get every name from a module.

If you want to separate interface, then you put everything into a `.mli` file (where you can define type signatures for functions). Don't worry about this for this class, but it's there if you want it.

#####September 24, 2014

#####A Complete Interpreter
The scanner and AST. 

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

Abstract Syntax Tree that represents arithmetic expressions.
```ocaml
type operator = Add | Sub | Mul | Div
type expr =
Binop of expr * operator * expr
| Lit of int
```
The scanner takes the input characters that you give it, and it gets structured into a tree through a parser. 

```ocaml
%{ open Ast %}
%token PLUS MINUS TIMES DIVIDE EOF
%token <int> LITERAL
%left PLUS MINUS
%left TIMES DIVIDE
%start expr
%type <Ast.expr> expr
%%
expr:
expr PLUS expr { Binop($1, Add, $3) }
| expr MINUS expr { Binop($1, Sub, $3) }
| expr TIMES expr { Binop($1, Mul, $3) }
| expr DIVIDE expr { Binop($1, Div, $3) }
| LITERAL { Lit($1) }
```

We will talk about OCaml Yak later. 

The parser takes the tokens from the scanner and builds the tree. `Binop` is a constructor. 

Once the input has been run through the scanner and parser, and an AST has been built, you need to take that AST and interpret it. If I have a 3, it's 3. If I have sum of 3 and 4, we should add them together. 

```ocaml
open Ast
let rec eval = function
Lit(x) -> x
| Binop(e1, op, e2) ->
let v1 = eval e1 and v2 = eval e2 in
match op with
Add -> v1 + v2
| Sub -> v1 - v2
| Mul -> v1 * v2
| Div -> v1 / v2
let _ =
let lexbuf = Lexing.from_channel stdin in
let expr = Parser.expr Scanner.token lexbuf in
let result = eval expr in
print_endline (string_of_int result) (* nice to see what we have, so print it out *)
```

When you run across a literal, it is what you want. A Binary operator is in the form `Binop(e1, op, e2)`. Need to evaluate `e1` and `e2` and then look at the operation, `op`.

This is a desktop calculator program. 

Compile using OCaml Yak, which we will talk about later. 

For project:
- Scanner
- AST
- Parser
- Interpreter 

You need your compiler to compile into something, not just the AST (compile to C, bytecode, C++, whatever). 

How can we overload things like `+`? One way is to make everything in the language a matrix. 

For parser, get the grammar right, pass it through OCaml yak to get a parser. 

#####So what are Compilers?
In this class, you are concerned with three languages:
- OCaml, which is what your compiler is written in
- Your input language
- Your output language

#####Language Processors
What we were shown earlier was a classical interpreter. It takes a source program as an input, and it takes some sort of input for the source program. The interpreter takes these two things, executes the source program on the input, and produces output. 

In the case of the calculator example, we get a textual interpretation of operations on integers.

What sorts of things are interpreters?
- A pocket calculator is an interpreter. There's an interpreter inside calculators that runs the calculator program within the calculator program. 
- Markdown translates to HTML. 
- Bash is an interpreter. Shells are implemented as interpreters. 

Look up 4004 Intel Micro Processor.
- Why did 4004 use an interpreter to write the calculator program? 
- The program was more concise if you wrote it in an interpreted language, rather than writing it in 4004 machine code. 
- This made the calculator slower, but the interpreted program was much smaller. 

Three Knee Jerk Reactions
- Add another type if you run into a problem
- When in doubt, add another level of indirection

Compiler is a middle man between source program and what gets executed in processor. 
Compiler looks at the program, and generates an equivalent program. 

Why use a compiler? Compiled code generally runs faster than interpreted code. Compilers optimize. Compilers catch mistakes, but interpreted programs run until it hits that mistake, it has to evaluate everything that leads to it. Lots of overhead goes away when using a compiler.

What are the advantages of an interpreter? 
- You don't have to wait for programs to compile
- Some interpreters are interactive / conducting tiny little experiments
- Supportability

Virtual Machines!

Source Program -> Compiler -> Bytecode -> Virtual Machine 

Java and Erlang have Virtual Machines. Virtual Machine is kind of like an interpreter that works on bytecode. Make the virtual machine portable. What's the drawback of VM? The VM is still an interpreter, so there might be a semantic mismatch between the bytecode and VM. But this runs slowly. 

Add another level of indirection:
Source Program -> Compiler -> Bytecode -> Just in time Compiler -> Machine Code

Just in time (JIT) compilation: compilation is done during run time :o 
There is always a trade off in JIT, time compiling down to bytecode makes a bit of a delay at the beginning. If you REALLY tune a JIT compiler, you might be able to get it as fast as a regular compiler. 

Bytecode is a series of instructions. To actually run those instructions, the easiest thing to do is be an interpreter. But there is still interpretation overhead. The idea of JIT, is that it grabs segments of the bytecode, and translates it into assembly instructions directly. 
