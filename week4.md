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
