###Programming Languages and Translators
----
#####September 15, 2014

OCaml

Review of map:

```ocaml
let rec map f = function
	[] -> []
	| head :: tail ->
		let r = f head in 
			r :: map f tail;;
```

#####Why OCaml?

- It is great for compilers. Different languages are good for different things.
- Generally functional languages are good for this. 
- If you're writing a lot of OCaml, you're doing it wrong. 
- Type system <3 Hindley-Milner
	- Catches a lot of programming mistakes.
	- If it manages to get through the compiler, than it will most likely just work.

Floating point arithmetic is different from regular arithmetic. This is denoted using a ```.``` after the operator. For example: ```3.0 +. 4.0```. 

No automatic promotions (from int -> double).
No explicit types, they are inferred. 

Declarations:

- `Let` declarations, binds functions and values to names.
- `Types`: algebraic data types.
- `Exceptions`: special types of type. 

Use CTRL-D or `#quit;;` to get out of OCaml repl.

#####Recursion

`else` is necessary in recursive statements. 

`let rec` allows for functions to call itself. 

Distinction between single equal sign and double equal sign:
`=` is structural equality.
`==` is physical equality (looks at where pointer points in memory).

There are no pointers in integers ("unboxed type").

Floating point numbers are represented as pointers under the heap. 

```ocaml
"a" = "a";;
-: bool = true

"a" == "a";;
-: bool = false
```

#####When in doubt in writing OCaml:

- create a new data type
- create a new function

#####Tuples

Tuples in OCaml is a very lightweight struct. Have a certain number of fields in them. Types of each tuple is fixed. 

Here is a tuple with type (Int, String).

```ocaml
#(42, Arthur)
-: int * string = (42, "Arthur")
```

Built in functions `fst` and `snd` take pairs (tuples with two elements) to access first or second item in tuple.

`_` is to refer to a value that exists but don't care what it is. 
For example, establish a pattern `(age, _)`, we just want to know the age of the person, but we don't care about his/her name. `age` should return `42` for `(42, "Arthur")`.

Tuples can have different types in them. Can't take a tuple and tack another field onto it. 

Lists are sort of the opposite.

#####Lists
Add stuff and shave things off arbitrarily. But every element in the list has to be the same type. 

The empty list is `[]`. 
Singleton list `[1]`.
Two element list `[1;2]`.

`[[]]` is a list of a list; it is a list that has an empty list as an element -> `'a list list`.	

What is type of `fst`, what about `snd`? 

Think of a list as a singly linked list.

```ocaml
fst : 'a * 'b -> 'a
```

```ocaml
snd : 'a * 'b -> 'b
```

`::` is very fast. Cons operator.
`@` (concat) is runs in linear time, list concatenation.

`head` gives you first element of a list.
 
`tail` gives you rest of list sans head.

Build list in reverse, then call `List.reverse`. List.reverse is linear time; concat is more expensive. 

In `if` `else` statements, `else` is compulsory.

#####Global and Local Value Declarations

The most common thing in OCaml is the `let` construct.

`let name = expr1 in expr2`. If you don't `let x in y`, then `x` will be available globally.

#####Functions

Function is just another type whose value can be defined with an expression. 

You can create unnamed functions (lambdas):

```ocaml
fun x -> x * x
- : int -> int -> <fun>
fun (x -> x * x) 5 ;;
- : int = 25
```

Bind a lambda to `square`.

```ocaml
let square = fun x -> x * x;;
```
#####Let / Static Scoping

OCaml picks up the values in effect where the function or expression is defined.

Think of what was actively in the scope when function was defined. 

Binding names is not assignment.

Variables are immutable. 

#####September 17, 2014
---
#####Functions

REVIEW

You don't have variables in the traditional sense.

Binding names is not assignment!

```ocaml
let a = 5 in
let b x = a + x in
let a = 42 in
b 0 ;;
```

`a` in `b x` is different from `a` in line three.

`let` is like function application.

```ocaml
let a = 3 in a + 2;;
``` 
is equivalent to 

```ocaml
(fun a -> a + 2) 3;;
```. Just syntactic sugar. First way is recommended. This is just to show these are equivalent functions.

Later in semester, we will learn about Lambda Calculus. <3

You can pass a function to a function in OCaml.

`rec` keyword is important; function is visible so it can refer to itself. 

Factorial: ```let rec fac n = if n < 2 then 1 else n * fac (n - 1);;```

`and` keyword allows for mutual recursion.

#####Some useful List functions 
`List.map`: apply a function to every element of the list. Use this if you want to do something to it without knowing where it is.

`List.fold_left`: Takes a function and an accumulator value. Apply a function to a partial result and an element of a result and pass the accumulator on. 
Example: List.fold_left (fun s e -> s + e) 0 [42; 17; 128];;

`List.iter`: use for something that has side effects, like printing.

Enumerating list example (for reversing a list):

```ocaml
# let (l, _) = List.fold_left
(fun (l, n) e -> ((e, n)::l, n+1)) ([], 0) [42; 17; 128]
in List.rev l;;
- : (int * int) list = [(42, 0); (17, 1); (128, 2)]
```

#####Pattern Matching
<3

Code snippet in Professor Edwards notes online.

```ocaml
# let xor p = match p
with (false, false) -> false
| (false, true) -> true
| ( true, false) -> true
| ( true, true) -> false;;
val xor : bool * bool -> bool = <fun>
# xor (true, true);;
- : bool = false
```

Can also be written as:

```ocaml
# let xor p = match p
with (false, x) -> x
| ( true, x) -> not x;;
val xor : bool * bool -> bool = <fun>
# xor (true, true)
- : bool = false
```

The compiler is smart enough to tell you when you've not included a case. Pattern matching might not be exhaustive. It can also figure out if you've done something redundant. 

Not enough cases:

```ocaml
# let xor p = match p
with (false, x) -> x
| (x, true) -> not x;;
Warning P: this pattern-matching is not exhaustive.
Here is an example of a value that is not matched:
(true, false)
val xor : bool * bool -> bool = <fun>
```

Redundant case:

```ocaml
# let xor p = match p
with (false, x) -> x
| (true, x) -> not x
| (false, false) -> false;;
Warning U: this match case is unused.
val xor : bool * bool -> bool = <fun>
```

`_` is used as a wildcard. I know there's something there, but I don't care what it is. Don't give it a name.

```ocaml
# let xor p = match p
with (true, false) | (false, true) -> true
| _ -> false;;
```	

Pattern Matching with Lists

```ocaml
# let length = function (* let length = fun p -> match p with *)
[] -> "empty"
| [_] -> "singleton"
| [_; _] -> "pair"
| [_; _; _] -> "triplet"
| hd :: tl -> "many";;
val length : ’a list -> string = <fun>
```
`hd` and `tl` are just arbitrary names. They could have been names other things, but it stands for head and tail (just a convention).

This is a polymorphic function, where polymorphic means "the code is written without mention of any specific type and thus can be used transparently with any number of new types".

Calculating the length of a list with pattern matching:

```ocaml
let length' _list = 
	[] -> 0
	| (x :: xs) -> 1 + length' xs;;
```

However, this isn't efficient because it is not tail recursive (needs O(n) stack space).

```ocaml
let length l =
let rec helper len = function
	[] -> len
	| _::tl -> helper (len + 1) tl
in helper 0 l
```

`helper` keeps track of at point in the iteration you are at. 

OCaml compiler knows how to optimize your pattern matches, don't be afraid. :)

LSB = Least Significant Bit

#####Directed Graphs

```ocaml
let edges = [
("a", "b"); ("a", "c");
("a", "d"); ("b", "e");
("c", "f"); ("d", "e");
("e", "f"); ("e", "g") ]
let rec successors n = function
	[] -> []
	| (s, t) :: edges -> if s = n then
							t :: successors n edges
						else
							successors n edges
```

** Look at slide 36 in Professor Edwards's OCaml notes for directed graph picture.

```ocaml
# successors "a" edges;;
- : string list = ["b"; "c"; "d"]
# successors "b" edges;;
- : string list = ["e"]
```

Here "a" is the source (s). "b", "c", and "d" are targets (t).
When s = n, we know that the s node is the node we want to find successors for (or targets). Now we know that the second argument of an edge that starts at "a" is a successor. 

We can also do this using filter (which is a more functional way of thinking):

```ocaml
let successors n edges =
	let matching (s,_) = s = n in
	List.map snd (List.filter matching edges)
```

Are you a tuple where your first element is the node name that I passed to you?