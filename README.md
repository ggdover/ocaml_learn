
# Setting up a project/code base

## dune

dune is a build-system for ocaml, making it easier to
handle dependencies for your ocaml program.

### Step-by-step Setup code base from scratch

1. ```dune init exec NAME``` or ```dune init e NAME```
* ```dune``` refers to the cli tool for the dune build-system
* ```init``` a dune command to initialize a new dune component
* ```exec``` first argument for the ```dune init``` command. Tells us the INIT_KIND. In this case we say that the INIT_KIND will be an executable. Another valid option is library.
You don't have to type exactly ```exec```, anything from ```e``` to more of the first letters making up the word ```executable``` will autocomplete to the INIT_KIND executable.
Same functionality for the INIT_KIND library
* ```NAME``` the name of the project/component you're initializing. Can be anything you find fitting.

2. ```dune build``` or ```dune b```
* Builds your source code

3. ```dune exec ./NAME.exe``` or ```dune exe ./NAME.exe```
* Run your executable

# Debugging and verify your code is correct

## Using utop

utop is a REPL (Read-Eval-Print-Loop) for ocaml.
There is a simpler interpreter that can be invoked
by calling ```ocaml```. For simplicity I have created
a alias in ```~/.bashrc``` such as ```alias ocaml="utop"```
so I invoke utop even if I type ```ocaml```.

### Using utop

When you've started utop you can just start writing ocaml code
directly into it. There are also utop commands that you can use,
which are prefixed with a ```#```.

All statements you write into utop, whether it's utop commands for
ocaml code needs a double semi-colon ```;;``` in order to make
sure that statement gets evaluated/run.

To find out more about available utop commands type ```#help;;```

### Load existing source code file (.ml) into utop

```#use "main.ml"```

# How to call a function

lets say you have a function defined like this
```
let read_file file =
    let ic = open_in file in
        try
            let line = input_line ic in
                let all_input = [] in
                    line :: all_input
        with e ->
            close_in_noerr ic;
            raise e;;
```

This function ```read_line``` is defined to take one argument called ```file``` which
is suppose to be a string to a file name.
So you can call this function like this: 
```
read_line "file.txt";;
```

# loading/including external libraries in your ocaml project

Another module/library is included into a source file (.ml) by
using the keyword ```open``` like this example: ```open digestif```.

* remember that the name of the library/module is case-sensitive when using ```open```

## The standard library (stdlib)

The OCaml standard library https://ocaml.org/api/Stdlib.html
Is always opened by default, meaning this module can be used
directly in any source file you write without using the
```open Stdlib``` statement.

## Loading external libraries

1. Include in your .ml file with ```open``` keyword
2. Add it to your dune configuration file (a file namned simply 'dune' in your project directory)
   list the library you want to load under the ```libraries``` attribute, like this:
   ```
   (executable
    (name hello)
    (libraries wall))
    ```
* remember that the name of the libraries are case-sensitive
3. The library might not be installed, so use the cli tool for ocamls source package manager: ```opam```. Here is an example on installing a library: ```opam install wall```.
* To list currently installed packages/libraries: ```opam list```
  * ```opam list``` defaults to ```opam install --installed```
* To list both uninstalled and installed packages: ```opam list -A``` or ```opam list --all```
* Run ```opam update``` to update list of available packages
* Run ```opam upgrade``` to upgrade installed packages to latest version

# the let-binding

Outside any other function/let-binding:<br>
```
let label arg1 arg2 =
    .....;;
```
Inside any other function/let-binding:
```
  let label = expression in
  ....
```

(Another way of think about the ```let ... = expression in``` is as a 
 nested/local/lambda function (A function declared within a function))

EDIT:

Actually maybe I got the difference between ```let ... = exp``` and ```let ... = exp in``` slightly wrong.
On second thought I think the made difference is that WITH the ```in``` keyword the body that the let-binding exists for is a expression that follow directly after the ```in``` keyword, where as WITHOUT the ```in``` keyword, the let-binding exists if you want to use it for multiple different expressions that follows.

# How to represent a function (let-binding) without arguments

```
let funcName () =
    ....;;
```
So in otherwords, we use the ```()``` where we normally write the arguments names

# What does ```let _ = exp``` mean?

It means, evaluate whats in the expression "exp"
but throw away the results. This is a little bit like the void return type on a function in C-like languages.

# Difference between ```let () = ...``` and ```let _ = ...```

```let () = exp```</br>
the expression 'exp' will not return any value (it will return 'unit' which is essentially equivalent to 'void' in c)

```let _ = exp```</br>
the value that is returned by expression 'exp' is ignored. Can also be used to ignore only a part of returned value. For example ```let (a_,b) = exp``` if 'exp' returns value of type 'pair'.

# OCaml file extensions

| Purpose                 | C      | Bytecode | Native code (OCaml) |
| ----------------------- |:-------| :--------| :-------------------|
| Source code             | *.c    | *.ml     | *.ml                |
| Header/Interface files  | *.h    | *.mli    | *.mli               |
| Object files            | *.o    | *.cmo    | *.cmx2              |
| Library files           | *.a    | *.cma    | *.cmxa3             |
| Binary programs         | prog   | prog     | prog.opt4           |
</br>

* *.mli file needs to be compiled before it's corresponding *.ml/source file

# Module and Submodule

Any source file with coupled interface file automatically makes a module in ocaml.
The name of the module is derived by the name of its source/interface file with first letter uppercase.
You can call/get values, functions and types from modules by typing <name of module>.<name of value, function or type>.
Alternativley you can write "open <name of module>" so can call the value, function or type directly, leaving out the name of the module. (Basically like doing "using namespace <name of namespace>" in C++)

You can also create a submodule by creating a module inside a file. Example:
```
module Hello = struct
  let message = "Hello"
  let hello () = print_endline message
end
let goodbye () = print_endline "Goodbye"
let hello_goodbye () =
  Hello.hello ();
  goodbye ()
```

lets say this is defined in a module namned Example (example .ml/.mli), then you can call the submodule like this:
```
let () =
  Example.Hello.hello ();
  Example.goodbye ()
```

# Submodule interface

Heres an example showing how you can define interface for your submodule, so you can restrict access to certain things in your submodule. In this example the let-binding (value) 'message' in made nuaccessable for outside modules:</br>
In the .mli file
```
module type Hello_type = sig
  val hello : unit -> unit
end
```
In the .ml file
```
module Hello : Hello_type = struct
  let message = "Hello"
  let hello () = print_endline message
end
```

While above way is how its normally done, but still just for the sake of it, heres a example showing how to do it so that the interface and implementation of submodule is in the same file
```
module Hello : sig
 val hello : unit -> unit
end = 
struct
  let message = "Hello"
  let hello () = print_endline message
end
  
(* At this point, Hello.message is not accessible anymore. *)
let goodbye () = print_endline "Goodbye"
let hello_goodbye () =
  Hello.hello ();
  goodbye ()
```

# OCaml interface (.mli) file

Function in .mli fil can look like this
```
val hello : unit -> unit
```
where the implementation (the function in the .ml file) looks like this:
```
let hello () = print_endline "Hello"
```

# type definitions

Defined like this

```
type date = { day : int; month : int; year : int}
```

and in the interface file it can be declared in 4 different ways:
* The type is ommited from .mli file (type is unaccessable from other modules)
* The type definition is copy-pasted into .mli file (type and all its fields is public)
* The type is made abstract, only its name is given (type is accessable but no fields are public (readable nor writeable))
* The type is accessable and its fields is read-only. Looks like this:
  * ```
    type date = private { ... }
    ```

# Functors, What are they

A functor is a module that is parametrized by another module, just like a function is a value which is parametrized by other values, the arguments.

# Functors, How to use them

Here is an example, where the module "Set" that exists in ocaml standard library has a functor "Make" that needs a module with two things: the type of elements, given as 't' and the comparison function given as 'compare', which it takes as a parameter:

```
# module Int_set = Set.Make (struct
                               type t = int
                               let compare = compare
                             end);;
```
Here is what the new module returned "Int_set" contains after this
```
module Int_set :
  sig
    type elt = int
    type t
    val empty : t
    val is_empty : t -> bool
    val mem : elt -> t -> bool
    val add : elt -> t -> t
    val singleton : elt -> t
    val remove : elt -> t -> t
    val union : t -> t -> t
    val inter : t -> t -> t
    val disjoint : t -> t -> bool
    val diff : t -> t -> t
    val compare : t -> t -> int
    val equal : t -> t -> bool
    val subset : t -> t -> bool
    val iter : (elt -> unit) -> t -> unit
    val map : (elt -> elt) -> t -> t
    val fold : (elt -> 'a -> 'a) -> t -> 'a -> 'a
    val for_all : (elt -> bool) -> t -> bool
    val exists : (elt -> bool) -> t -> bool
    val filter : (elt -> bool) -> t -> t
    val partition : (elt -> bool) -> t -> t * t
    val cardinal : t -> int
    val elements : t -> elt list
    val min_elt : t -> elt
    val min_elt_opt : t -> elt option
    val max_elt : t -> elt
    val max_elt_opt : t -> elt option
    val choose : t -> elt
    val choose_opt : t -> elt option
    val split : elt -> t -> t * bool * t
    val find : elt -> t -> elt
    val find_opt : elt -> t -> elt option
    val find_first : (elt -> bool) -> t -> elt
    val find_first_opt : (elt -> bool) -> t -> elt option
    val find_last : (elt -> bool) -> t -> elt
    val find_last_opt : (elt -> bool) -> t -> elt option
    val of_list : elt list -> t
    val to_seq_from : elt -> t -> elt Seq.t
    val to_seq : t -> elt Seq.t
    val add_seq : elt Seq.t -> t -> t
    val of_seq : elt Seq.t -> t
  end
```
If the module we are suppose to pass as parameter is already prepared, its alot easier. Like in this example when we pass in the module "String" that is already defined in the ocaml standard library:
```
# module String_set = Set.Make (String);
```

# Functors, how to define them

A functor with one argument can be defined like this:
```
module F (X : X_type) = struct
  ...
end
```
where ```X``` is the module that will be passed as argument, and ```X_type``` is its signature, which is mandatory.

You can also constrain what the signature of the returned module should be. Example:
```
module F (X : X_type) : Y_type =
struct
  ...
end
```
or if you specify it in the .mli file instead
```
module F (X : X_type) : Y_type
```
If you want more examples on how to write functors you can look at the ```set.ml/.mli``` and ```map.ml/.mli``` modules/files in the ocaml standard library.

# Functor, Module inclusion

Let's say we feel that a function is missing from the standard ```List``` module, but we really want it as if it were part of it. In an ```extensions.ml``` file, we can achieve this effect by using the ```include``` directive:

```
# module List = struct
    include List
    let rec optmap f = function
      | [] -> []
      | hd :: tl ->
         match f hd with
         | None -> optmap f tl
         | Some x -> x :: optmap f tl
  end;;
```
which results in the following returned module:
```
module List :
  sig
    type 'a t = 'a list = [] | (::) of 'a * 'a list
    val length : 'a list -> int
    val compare_lengths : 'a list -> 'b list -> int
    val compare_length_with : 'a list -> int -> int
    val cons : 'a -> 'a list -> 'a list
    val hd : 'a list -> 'a
    val tl : 'a list -> 'a list
    val nth : 'a list -> int -> 'a
    val nth_opt : 'a list -> int -> 'a option
    val rev : 'a list -> 'a list
    val init : int -> (int -> 'a) -> 'a list
    val append : 'a list -> 'a list -> 'a list
    val rev_append : 'a list -> 'a list -> 'a list
    val concat : 'a list list -> 'a list
    val flatten : 'a list list -> 'a list
    val iter : ('a -> unit) -> 'a list -> unit
    val iteri : (int -> 'a -> unit) -> 'a list -> unit
    val map : ('a -> 'b) -> 'a list -> 'b list
    val mapi : (int -> 'a -> 'b) -> 'a list -> 'b list
    val rev_map : ('a -> 'b) -> 'a list -> 'b list
    val filter_map : ('a -> 'b option) -> 'a list -> 'b list
    val concat_map : ('a -> 'b list) -> 'a list -> 'b list
    val fold_left : ('a -> 'b -> 'a) -> 'a -> 'b list -> 'a
    val fold_right : ('a -> 'b -> 'b) -> 'a list -> 'b -> 'b
    val iter2 : ('a -> 'b -> unit) -> 'a list -> 'b list -> unit
    val map2 : ('a -> 'b -> 'c) -> 'a list -> 'b list -> 'c list
    val rev_map2 : ('a -> 'b -> 'c) -> 'a list -> 'b list -> 'c list
    val fold_left2 : ('a -> 'b -> 'c -> 'a) -> 'a -> 'b list -> 'c list -> 'a
    val fold_right2 :
      ('a -> 'b -> 'c -> 'c) -> 'a list -> 'b list -> 'c -> 'c
    val for_all : ('a -> bool) -> 'a list -> bool
    val exists : ('a -> bool) -> 'a list -> bool
    val for_all2 : ('a -> 'b -> bool) -> 'a list -> 'b list -> bool
    val exists2 : ('a -> 'b -> bool) -> 'a list -> 'b list -> bool
    val mem : 'a -> 'a list -> bool
    val memq : 'a -> 'a list -> bool
    val find : ('a -> bool) -> 'a list -> 'a
    val find_opt : ('a -> bool) -> 'a list -> 'a option
    val find_map : ('a -> 'b option) -> 'a list -> 'b option
    val filter : ('a -> bool) -> 'a list -> 'a list
    val find_all : ('a -> bool) -> 'a list -> 'a list
    val partition : ('a -> bool) -> 'a list -> 'a list * 'a list
    val assoc : 'a -> ('a * 'b) list -> 'b
    val assoc_opt : 'a -> ('a * 'b) list -> 'b option
    val assq : 'a -> ('a * 'b) list -> 'b
    val assq_opt : 'a -> ('a * 'b) list -> 'b option
    val mem_assoc : 'a -> ('a * 'b) list -> bool
    val mem_assq : 'a -> ('a * 'b) list -> bool
    val remove_assoc : 'a -> ('a * 'b) list -> ('a * 'b) list
    val remove_assq : 'a -> ('a * 'b) list -> ('a * 'b) list
    val split : ('a * 'b) list -> 'a list * 'b list
    val combine : 'a list -> 'b list -> ('a * 'b) list
    val sort : ('a -> 'a -> int) -> 'a list -> 'a list
    val stable_sort : ('a -> 'a -> int) -> 'a list -> 'a list
    val fast_sort : ('a -> 'a -> int) -> 'a list -> 'a list
    val sort_uniq : ('a -> 'a -> int) -> 'a list -> 'a list
    val merge : ('a -> 'a -> int) -> 'a list -> 'a list -> 'a list
    val to_seq : 'a list -> 'a Seq.t
    val of_seq : 'a Seq.t -> 'a list
    val optmap : ('a -> 'b option) -> 'a t -> 'b t
  end
```
to then make sure that the ```List``` module that already exists in ocaml standard library is overridden by the modified one in ```extensions.ml``` you simple do ```open Extensions```:
```
open Extensions
...
List.optmap ...
```

# Linked lists

To represent a empty list you write ```[]```</br>
The following show different ways to generate the exact same list:
```
[1; 2; 3]
1 :: [2; 3]
1 :: 2 :: [3]
1 :: 2 :: 3 :: []
```
as you can see, the ```::``` works a bit like a operator for concatenation, in fact it's actual name is ```cons operator```. We'll see later how it comes in handy in pattern matching on lists.

# Type of linked lists (its elements)

You define which elements a list holds by writing like this ```int list```. In Ocaml lists can only hold elements of one type, there is however a special type which is typed it as ```'a``` which is polymorphic to represent any type. So to define a list with this type you would do ```'a list```. Here is an example with the function ```length``` that takes list of the type ```'a```:
```
List.length : 'a list -> int
```

# Structures

In C and C++ theres the concept of ```struct```, the simplest equivalent to this in OCaml is a tuple such as (3,4) which would be equivalent to a C struct as:
```
struct pair_of_ints {
  int a, b;
};
```
Unlike lists, tuples in OCaml can contain elements of different types, for example: ```(3, "hello", 'x')```, which has the type: int, string, char.</br>
</br>
A slightly more complex alternative way of writing C struct is to use a ```record```. Records allow you to name the elements. Tuples don't let you name the elements, but instead you have to remember the order in which they appear. Heres an example of a ```record```:
```
# type pair_of_ints = { a : int; b : int };;
```
this defines the type, and here is how we create objects of this type:
```
# { a=3; b=5 };;
```
NOTE! Ocaml won't let you leave some fields of you structure undefined. See following example:
```
# type pair_of_ints = { a : int; b : int };;
type pair_of_ints = { a : int; b : int; }
# {a=3; b=5};;
- : pair_of_ints = {a = 3; b = 5}
# {a=3};;
Error: Some record fields are undefined: b
```

# Variants (Qualified unions and enums)

## Qualified unions

The difference between Qualified/Tagged union and regular union (C only have regular unions) is that Qualified/Tagged union keeps track of which type it has been set to represent, in C it's up to the developer to ensure that the union object is used correctly.

The way we write unions (Qualified/Tagged unions that is) is with the keyword ```type```. Here is an example:
```
# type foo =
    | Nothing
    | Int of int
    | Pair of int * int
    | String of string;;
```
```foo``` is the type name of the union and following each ```|``` token is whats called a constructor and can be called anything, first letter should be capitalized (Compring to C unions, a constructor is equivalent to name of a field/element). If the constructor can be used to define a value, it's followed by the ```of``` keyword and then the value to be stored is described.</br>
To actually create objects/things of this ```type``` (qualified union) you would write:
```
Nothing
Int 3
Pair (4, 5)
String "hello"
...
```
Each of these expressions has the type ```foo```

## Enum

By extending ````type``` in OCaml you can also easily represent a enum. See following example where we show how a C enum can be represented in OCaml:
```
enum sign { positive, zero, negative };
```
and to get the equivalent enum in OCaml we write
```
type sign = Positive | Zero | Negative
```

## Recursive variants

Variants can be recursive in OCaml, and one common use for this is to define tree structures. This is where the expressive power of functional languages come into their own:
```
# type binary_tree =
    | Leaf of int
    | Tree of binary_tree * binary_tree;;
```
And here is how you would use the type to create some binary trees.
```
Leaf 3
Tree (Leaf 3, Leaf 4)
Tree (Tree (Leaf 3, Leaf 4), Leaf 5)
Tree (Tree (Leaf 3, Leaf 4), Tree (Tree (Leaf 3, Leaf 4), Leaf 5))
```

## Parameterized variants

The binary tree in the previous section has integers at each leaf, but what if we wanted to describe the shape of a binary tree, but decide exactly what to store at each leaf node later? We can do this by using a parameterized (or polymorphic) variant, like this:
```
# type 'a binary_tree =
    | Leaf of 'a
    | Tree of 'a binary_tree * 'a binary_tree;;
```

This is a general type. The specific type which stores integers at each leaf is called ```int binary_tree```. Similarly the specific type which stores strings at each leaf is called ```string binary_tree```. In the next example we type some instances into the top-level and allow the type inference system to show the types for us:

```
# Leaf "hello";;
- : string binary_tree = Leaf "hello"
# Leaf 3.0;;
- : float binary_tree = Leaf 3.
```

# Pattern matching (on datatypes)

So one Really Cool Feature of functional languages is the ability to break apart data structures and do pattern matching on the data. This is again not really a "functional" feature - you could imagine some variation of C appearing which would let you do this, but it's a Cool Feature nonetheless.

Let's write a function which prints out Times (Value "n", Plus (Value "x", Value "y")) as something more like n * (x + y). Actually, I'm going to write two functions, one which converts the expression to a pretty string, and one which prints it out (the reason is that I might want to write the same string to a file and I wouldn't want to repeat the whole of the function just for that).

```
# let rec to_string e =
    match e with
    | Plus (left, right) ->
       "(" ^ to_string left ^ " + " ^ to_string right ^ ")"
    | Minus (left, right) ->
       "(" ^ to_string left ^ " - " ^ to_string right ^ ")"
    | Times (left, right) ->
       "(" ^ to_string left ^ " * " ^ to_string right ^ ")"
    | Divide (left, right) ->
       "(" ^ to_string left ^ " / " ^ to_string right ^ ")"
    | Value v -> v;;
val to_string : expr -> string = <fun>

# let print_expr e =
    print_endline (to_string e);;
val print_expr : expr -> unit = <fun>
```

Here's the print function in action:

```
# print_expr (Times (Value "n", Plus (Value "x", Value "y")));;
(n * (x + y))
- : unit = ()
```

The general form for pattern matching is:

```
match value with
| pattern    ->  result
| pattern    ->  result
  ...
```

## Pattern matching guards (the 'when' keyword)

You can add what are known as guards to each pattern match. A guard is the conditional which follows the when, and it means that the pattern match only happens if the pattern matches and the condition in the when-clause is satisfied.

```
match value with
| pattern  [ when condition ] ->  result
| pattern  [ when condition ] ->  result
  ...
```

Here is an example where the 'when' clause is used for a "Factorization" function made using pattern matching:

```
# let factorize e =
    match e with
    | Plus (Times (e1, e2), Times (e3, e4)) when e1 = e3 ->
       Times (e1, Plus (e2, e4))
    | Plus (Times (e1, e2), Times (e3, e4)) when e2 = e4 ->
       Times (Plus (e1, e3), e4)
    | e -> e;;
val factorize : expr -> expr = <fun>
# factorize (Plus (Times (Value "n", Value "x"),
                   Times (Value "n", Value "y")));;
- : expr = Times (Value "n", Plus (Value "x", Value "y"))
```

## Pattern matching, warning covering pattern possibilities

OCaml is able to check at compile time that you have covered all possibilities in your patterns. Look at following example, first we have a ```type``` called "expr" covering several types:

```
# type expr = Plus of expr * expr      (* means a + b *)
            | Minus of expr * expr     (* means a - b *)
            | Times of expr * expr     (* means a * b *)
            | Divide of expr * expr    (* means a / b *)
            | Product of expr list     (* means a * b * c * ... *)
            | Value of string          (* "x", "y", "n", etc. *);;
type expr =
    Plus of expr * expr
  | Minus of expr * expr
  | Times of expr * expr
  | Divide of expr * expr
  | Product of expr list
  | Value of string
```

Lets see what happens when we try and pattern match against this type in a function we call "to_string". Notice how OCaml reports a warning because the "Product" constructor was not handled in the pattern matching:

```
# let rec to_string e =
    match e with
    | Plus (left, right) ->
       "(" ^ to_string left ^ " + " ^ to_string right ^ ")"
    | Minus (left, right) ->
       "(" ^ to_string left ^ " - " ^ to_string right ^ ")"
    | Times (left, right) ->
       "(" ^ to_string left ^ " * " ^ to_string right ^ ")"
    | Divide (left, right) ->
       "(" ^ to_string left ^ " / " ^ to_string right ^ ")"
    | Value v -> v;;
Warning 8: this pattern-matching is not exhaustive.
Here is an example of a case that is not matched:
Product _
val to_string : expr -> string = <fun>
```

## List of different patterns to match

The following is the best list I have found so far, on the possible patterns that one can match against when doing "Pattern matching":
```
Patterns:
  | Pair (x,y) -> variant pattern
  | { field = 3; _ } -> record pattern
  | head :: tail -> list pattern ('head' and 'tail' can have other names. Common is 'h' and 't' instead)
  | [1;2;x] -> list pattern ('[]' to match empty list)
  | (Some x) as y -> with extra binding
  | (1,x) | (x,0) -> or-pattern
  | exception exn-> try & match
```

# The "Option" type (The "Maybe" type)

## TODO:
Write up some information around this type. I think this along with the other parts about ocaml's static type nature that is the biggest hurdle before I can get more comfortable and eventually fluent in OCaml.</br></br> 

Read more about it in the following links:
* https://www.cs.princeton.edu/~dpw/courses/cos326-12/lec/02c-unit-option.pdf

# The "Core" Ocaml library

## TODO:
Write about what you get from this library that feels like its lacking abit in ocamls built-in module "Pervasives" which is automatically opened in any ocaml code you write. More info here: https://ocaml.org/releases/4.02/htmlman/libref/Pervasives.html

Here is also a link to the official github for the Core library/Module: https://github.com/janestreet/core

