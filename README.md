# An Optional Type System for JS  

# Introduction

 
This is [stage-0](https://tc39.github.io/process-document/) proposal to add Optional Types to [JS](https://github.com/tc39/ecma262/) and bake them into the open web platform. 
 
For prior art (e.g. other languages), alternatives (e.g. gradual sound typing, macros and the status quo), challenges and future work we encourage you to start with the [FAQ](FAQ.md).
 
In the last 10 years, large engineering teams have developed type systems for JavaScript through a variety of preprocessors, transpilers and code editors to scale large codebases.
 
Most notably, TypeScript (Microsoft) has gained a massive amount of external adoption and is used as the foundation and starting point of this proposal. Flow (Facebook) and the Closure Compiler (Google) have also also been used extensively (although more confined to their respective internal audiences) and their experience informs the design of this proposal too.
 
Fortunately, these transpilers share a substantial amount of commonality (syntactically and semantically) and we leverage that as much as we can. Notably, at the core of these transpilers and hence at the center of this proposal is an **Optional Type System**: types are used at development-time (e.g. on preprocessors, IDEs, code editors, browser developer tools/mode/debuggers, etc) and are erased at production-time (e.g. interpretation of javascript for real users in production).
 
By baking an optional type system into the standard javascript language, we enable browsers to include type checking in developer tools making type systems (a) more broadly accessible to web developers and (b) more powerful. An in-browser typechecker works in conjunction with current transpilers who’d have a richer, typed and interoperable (between transpilers) compilation target to use and the added ability to catch type errors at debugging-runtime (see [dart’s checked mode](https://www.dartlang.org/articles/language/optional-types#checked-mode)) in addition to errors caught statically. It also works well in conjunction with minifiers/optimizers which can strip the types prior to deployment or safely assume that they’ll be ignored at production-runtime.

![In-browser type checker](browser.png)

As a general rule of thumb, at its current stage, this is a proposal for adding “an” optional type system to JavaScript more so than adding “this” specific one. To get the ball rolling, we start with a proposal with some broad strokes (to give an idea of feasibility and desirability), but we are looking to form/identify a working group to collectively define the specifics of the type system at later stages of the process (TC39 members, typescript/closure/flow experts, type system experts, developer tooling experts, etc).
 
With that, at this stage 0, we are generally looking to:

* Gather early feedback/recommendations/suggestions/validation on the general direction, motivation and approach (e.g. sanity check coverage and trade-offs in FAQ)
* Identify/invite a group of experts interested in the topic (hint: if you are reading this right now chances are you are one of them :)) to form an informal working group.

To kick things off we invite you to take a look at the following strawman and the open design questions section and help us collectively shape what this looks like.

# Strawman

To a large extent, like it was said earlier, this is more of a proposal for "**a**" typesystem (rather than "**this**" typesystem) and a process to get us there.
 
We are collecting the set of discussion points in the “open design questions” section (e.g. subset of the minimally-maximal set, sequencing of features, nominal versus structural classes, generics, etc).
 
As a rule of thumb, we annotate the most tentative points with a [***](FAQ.md).
 
As the design choices find convergence in the “open design questions” section we’ll pull them into this section, so hopefully this document should converge to our collective choices.
 
With that in mind, at the core of the existing type systems - and hence of this strawman proposal - is **Optional Typing** [***](FAQ.md): type checking that is processed at development-time/compilation-time/debugging-time only and erased at production-time.

Effectively, most of the information in this proposal is applicable to development-time tools (e.g. code editors, IDEs, compilers and developer tools in browsers). To the extent that new grammar/syntax is introduced and is to be erased rather than rejected, the semantics of interpretation of javascript in production-mode remain unchanged.
 
With that foundation in place, the MVP type system is composed of:

* A taxonomy of Types
* Typechecking Rules
* Subtyping Rules
* Syntax

We start by introducing the basic type taxonomy, the high level idea of the subtyping rules and we then later introduce the syntax and give a lot of examples.

# Type Taxonomy

We could really use some help with the art of sequencing and MVP-ing here (please see section on "open design questions"), but here is our current model for what could form a solid/strong foundation to be built upon.

* The **Any** type: represents any javascript value
* **Primitive types**: number, boolean, string and symbol
* The **Null** and **Undefined** types
* **Built-in Object Types**: the built-in standard object types (Object, Function, Error, Array, Iterable, Promise, Number, Math, Date, Map, etc)
* **User-defined Object Types**: structural types, interfaces, classes, functions arrays
* **Union Types** (***): types that represent values that may have one of several distinct representations.

TODO(goto, domenic): A major missing piece is the type definitions for the standard library. We should also say something about the type definitions for the web platform, since that's JS's real standard library.

# Typechecking Rules

For each expression in the program, we need to define a rule that checks if it is well typed, based on the types of its operands. For example, in a function-call expression, it is a type error if the subexpression in the callee position does not have a function type. For the minus expression, it is a type error if its operands do not have the number type.
 
For the time being, we do not define the type checking rules in this document. We expect them to be similar to the rules of the existing type systems (TypeScript, Flow, and Closure Compiler). Once there is consensus on the high-level design principles proposed here, we can define the type-checking rules in detail.

# Subtyping Rules

The design space for subtyping rules is big, and we have some picked semantics below but we understand these things are subject to change.

* The **Any** type to be a subtype and supertype of all types.
* Classes to be **nominally typed** ***.
* Objects types, interfaces and functions to be **structurally typed**.
* To be safe to use objects with extra properties ([Width subtyping](https://flow.org/en/docs/lang/width-subtyping/)) in a position that is annotated with a specific set of properties.
* Function parameters to be **contravariant** and function return values to be **covariant** ([Type variance](https://flow.org/en/docs/lang/variance/)).
* Arrays to be **covariant** ***.

In the next section we’ll introduce the new syntax and with that give a lot of examples on these rules.

# Syntax

You associate types with variables using newly introduced syntax:

```javascript 
// Any and Primitive Types
// Variable declarations.
let a: number = 1;
let b: number = 2; // Works!
let c: number = "3"; // Error!
let d: number = NaN; // Works!
let e: number = Infinity; // Works!
 
// TODO(goto): craft really good examples for Any
```
 
With Union Types you can represent types that can have one of several distinct representations. A value of union type of A | B is a value that is either of type A or type B.

```javascript 
// Union Types
let x: string | number; 
let test: boolean = true; 
x = "hello"; // Ok 
x = 42; // Ok 
x = test; // Error, boolean not assignable 
x = test ? 5 : "five"; // Ok 
x = test ? 0 : false; // Error, number | boolean not assignable
```
 
Functions are extended to type parameters and return values:
 
```javascript
// Functions
function concat(a: string, b: string): string {
  return a + b;
} 
concat("foo", "bar"); // Works! 
concat(true, false); // Error!
let c: number = concat("foo", "bar"); // Error!
 
// TODO(goto): come up with a better example.
```
 
It is safe to substitute a function that accepts a more general argument (contravariant inputs) and returns a more specific value (covariant outputs).

``` 
// Function subtyping
function fetch(
  url: string, 
  callback: (result: string) => number | undefined) {
  // ...
}
 
function a(result: string | undefined) : number | undefined {}
function b(result: string) : number {}
function c(result: boolean) : number | undefined {}
function d(result: string) : number | undefined | null {}
 
fetch(“data.json”, a); // Works
fetch(“data.json”, b); // Works
fetch(“data.json”, c); // Error
fetch(“data.json”, d); // Error
```
 
Types are [non-nullable by default](https://flow.org/en/docs/types/maybe/) ***, and types can be made nullable by using explicitly the union type. Function parameters and object properties are required by default and can be made optional by using union with undefined.

```javascript
// Optional and Nullable Types
function a(value: number | undefined | null) {
  // …
} 
a(42); // Works! 
a(); // Works! 
a(undefined); // Works! 
a(null); // Works! 
a("42"); // Error!
```
 
Object literals give you the ability to create structural types:

```javascript
// Object Literals
// object type literals give you the ability to add types to objects!
var person: { 
  firstName: string,
  lastName: string,
  age: number | undefined
} = { 
  firstName: "Sam",
  lastName: "Goto",
  // age is optional! phew!
};
```
 
Interfaces provide the ability to name structural types. Interfaces have no run-time representation and are purely a compile-time construct. Class syntax is extended so that classes can declare the interfaces they implement.
 
```javascript
Examples
interface Bar {
  hello(a: number): number;
}
 
class Foo implements Bar {
  bar(a: number) : number {
    return 0;
  }
}
 
// TODO(goto): find a better example here.
```
 
To create an Array type you can use Array<Type> where Type is the type of the elements of the array. You can use Array literals (e.g. string[] is equivalent to Array<string>) as a convenient shorthand. For example:

```javascript
// Examples of Array Type Usage
(example from flow’s array types and typescript’s)
let arr1: Array<boolean> = [true, false, true]; 
let arr2: Array<string> = ["A", "B", "C"];
let arr3: Array<any> = [1, “two”, false];
let arr4: Array<number|string>  = [1, 2, 3];
let a: string[] = ["hello", "world"];

// Arrays are covariants.
 
let arr5: Array<any> = arr1; // Works
let arr6: Array<string> = arr4; // Fails
arr4 = arr2; // Works
 
// Since the 10th element isn’t set, this returns undefined at
// runtime. The type system assumes bounded access though, allowing
// val to be of type string rather than string | undefined.
let val: string = arr2[10]; // Works
```
 
Note that this notation is a special case of of generics, so things like Array<? extends Type> aren’t yet allowed until generics are introduced (see section on Future Work in the [FAQ](FAQ.md)).
