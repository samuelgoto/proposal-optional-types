# FAQ

# Table of contents

* [Terminology](#terminology)
* [Where is this at?](#where-is-this-at)
* [Alternatives considered](#alternatives-considered)
    * [Status Quo](#status-quo)
    * [Sound Gradual Typing](#sound-gradual-typing)
    * [Decorators](#decorators)
    * [Macros](#macros)
* [Considerations](#considerations)
* [Prior art](#prior-art)
    * [Other languages](#other-languages): ([dart](#dart), [python](#python))
    * [TC39 discussions](#tc39-discussions)
* [The Type System](#the-type-system)
    * [Strategy](#strategy)
    * [Sequencing](#sequencing)

# Terminology
 
* **Optional Type System**: A typesystem where (a) run-time semantics are independent of type system and (b) type annotations are optional ([@bracha.org](http://bracha.org/pluggable-types.pdf)).
* **Transpilers**: Tools that consume JavaScript, perhaps with some non-standard syntactic constructs, and lower it to more "vanilla" JavaScript, e.g., by turning ES6 to ES5, or by removing type annotations. In this context, TypeScript, Flow, and Closure Compiler.
* **Coding-time**: the development stage where code is being edited, typically in a code editor (e.g. emacs, vim, notepad) or in an IDE (e.g. eclipse, intellij, visual studio)
* **Compilation-time**: the development stage where code is being compiled/minified (e.g. babel, tsc, closure compiler, etc).
* **Debugging-time**: the development stage where code is being debugged in a runtime scenario (e.g. browser developer tools). Expected to have more runtime information (e.g. profiling) at the cost of performance.
* **Development-time**: the union of coding-time, compilation-time and debugging-time where the programmer iterates on their code.
* **Production-time**: the deployment stage where code that is highly optimized gets used by real users and is expected to be as efficient as possible (e.g. chrome, firefox, internet explorer, safari, etc).
* **Checked-mode**: a debugging-time environment where runtime type errors are checked (see [dart’s checked mode](https://www.dartlang.org/articles/language/optional-types#checked-mode)).
* **Production-mode**: a production-time environment where type errors are erased and performance isn’t degraded.
* **Type-checker**: a development-time tool that performs type checking comforming to a specification (e.g. preprocessors, IDEs, code editors, browser developer tools/mode/debuggers, etc).
* **Interpreter**: a development-time and production-time tool that interprets Javascript and, besides erasing types, remains otherwise unchanged to be comformant with the standard.
* **In-browser type checker**: a hypothetical type-checker added to existing in-browser developer tools (e.g. [safari's type profiler](https://webkit.org/blog/3846/type-profiling-and-code-coverage-profiling-for-javascript/)).
* **MVP**: the minimum-viable-product, also known as [maximally-minimal](http://wirfs-brock.com/allen/files/papers/standpats-asianplop2016.pdf).


# Where is this at?

This is, by far, what you’d call "drafty", "strawman" or Stage 0 :)
 
We still have more questions than answers, but we think the direction is right.
 
We have a general intuition that this is a highly desirable feature, by the levels of adoption and maturity of existing type systems (some in place for over 10 years).
 
We also think we have collected solid data points (in the industry with interpreters and transpilers and in academia) to encourage us to look at Optional Type Systems (as opposed to [sound gradual types](#sound-gradual-typing)).
 
We find more commonalities than disparities in existing production-ready type systems for JavaScript (typescript, flow and closure), but we acknowledge that there are disparities and that the devil is in the details.
 
We find the trend of [adding types to dynamic languages](#other-languages) (e.g. Python, and Dart) encouraging. 
 
We don’t feel strongly about the specifics of the type system, in as much as we feel that one should exist :) To get the ball rolling, we start with a [strawman proposal](README.md#strawman) and hope we all collectively take it from here.
 
We don’t think this is not a [novel idea](#prior-art). We believe that the **circumstances** are different and more favorable to [revisiting](#tc39-discussions) the subject, more specifically:

* the level of adoption of TypeScript
* industry experiments with [sound gradual typing](https://groups.google.com/forum/#!msg/strengthen-js/ojj3TDxbHpQ/5ENNAiUzEgAJ) and
* precedence in [other languages](#other-languages).

# Alternatives considered

## Status quo?

What’s wrong with leaving things as is?
 
The biggest challenge we face keeping the status quo is twofold:
 
* Reach/power of type checkers for web developers
* Fragmentation of the ecosystem
 
First, a javascript-based type system increases the chances we’ll see it adopted by [in-browser developer tools](#terminology) (which already process javascript heavily), significantly decreasing the friction to access/use them.
In addition to access, an [in-browser type checker](#terminology) can increase the extent to which runtime errors are checked at [debugging-time](#terminology) with in-browser debuggers (as an example of what can be made possible, see [dart’s runtime checked mode](https://www.dartlang.org/articles/language/optional-types#checked-mode)).
 
Secondly, libraries written in one of the existing type system [transpilers](#terminology) are not interoperable with the others: large teams needing to depend on or export libraries end up in an incohesive and incoherent compilation environment.
 
With the [extensible web manifesto](https://extensiblewebmanifesto.org/) process in mind, our intuition is that we currently have a good balance between maturity/convergence/adoption and usefulness between the main polyfills/transpilers/compilers that is worth baking into the language the more popular/converged features.

## Sound gradual typing?

An alternative to (unsound) optional typing is (sound) gradual typing, as it exists, for instance, in Typed Racket. When the type checker cannot prove that a type is correct, runtime checks are inserted. There are three main drawbacks with this approach:
 
* **Performance**. The runtime checks significantly degrade performance; slowdowns of 2X or more are common (see [Racket paper](http://www.ccs.neu.edu/racket/pubs/popl16-tfgnvf.pdf), [Safe TypeScript](http://www.cs.umd.edu/~aseem/safets-tr.pdf), [JS strong mode](https://groups.google.com/forum/#!msg/strengthen-js/ojj3TDxbHpQ/5ENNAiUzEgAJ)). Quoting from the V8 team’s [analysis](https://groups.google.com/forum/#!msg/strengthen-js/ojj3TDxbHpQ/5ENNAiUzEgAJ): “In particular, we no longer believe that requiring type soundness _by default_ can ever work successfully in JavaScript. It may still be possible to provide sound types as an _opt-in_, though. But more on this at some other time.”.
* **Complexity**. Inevitably, a highly-expressive language such as JS has features that are hard to typecheck in a sound way. Gradual type systems have handled complex language features by introducing complicated types (e.g., [Dependent JavaScript](https://github.com/ravichugh/djs), [variable-arity polymorphism](http://www.ccs.neu.edu/racket/pubs/esop09-sthf.pdf)). We believe that keeping the type system simple is crucial for adoption.
* **Changes to the runtime semantics**. With an optional type system, a JS developer that does not want to use types is free to do so. Their code can call into typed code without changes. But since gradual typing requires runtime checks, it becomes pervasive. Besides the impact to developers, gradual typing requires significant changes to VMs, making it harder to adopt.
 
Overall, sound gradual typing has so far only existed in a research setting, and it is not clear whether it can be made practical. The only type systems for dynamic languages that have succeeded in industry are optional type systems.

## Decorators?
 
NOTE(domenic): in particular with additional decorator positions (such as functions and arguments) you could create most of a type system with decorators, with either runtime or AOT checks. It's a bit of a stretch to put them on variable declarations... but still.
 
TODO(goto): come up with a few examples and explore them a bit more.

## Macros?
 
One of the most promising/interesting approach to be explored is introducing macros (hint, not the #define-like macros, but [lisp-like macros](sweetjs.org)) to JavaScript: the parsing time ability to meta-program (code that changes code).
 
Macros are great for defining little languages within a language that are suited for specific problems (see the [sweetjs](sweetjs.org) project). However, it is not clear what the advantages are of doing a type system using macros:
 
* Types are a core part of a language, so it is good to have a single type system, specified in one place. Macros can allow several incompatible type systems to be created, which increases fragmentation.
* Types can easily be stripped by preprocessors, so it is not necessary to use a macro expander to remove types. Moreover, it is cheaper to strip types on the server, rather than rely on runtime macro expansion in the browser.

# Considerations

## Does this need to be part of the language?

Our intuition is that it does, for the [same reasons](#status-quo) that keeping the status quo is not ideal:

* an [in-browser type checker](#terminology) can substantially increase the reach of type systems and their ability to catch [debugging-time](#terminology) errors, and
* browsers are currently heavily invested in javascript and its tooling ecosystem.

We also think that tc39 **is** the [most natural venue](#is-tc39-the-right-venue) to standardize a typechecker.

## Does this grow the language unnecessarily?

Guy Steele does a really good job at describing the challenges of [growing a language](https://www.youtube.com/watch?v=_ahvzDzKdB0). Mark Miller does as well in his excellent article [The Tragedy of the Common Lisp, or, Why Large Languages Explode](https://mail.mozilla.org/pipermail/es-discuss/2015-June/043307.html).
 
The fundamental challenge here is that introducing new syntax comes with a cost and the cost gets compounded with time: users learning the language need to be introduced to new (and complicated!) concepts and existing users eventually get in contact with the features while reading/embedding external libraries.
 
There is a desire to keep the language small enough that one can understand it completely.
	
As Mark also acknowledges in the article, the introduction of classes in ES6 came with  the cost of growing the language but was justified by its massive benefits.
 
Even though types increase complexity by increasing the size of the language, they reduce complexity in other ways. First, programmers often code with types in mind, even in dynamic languages. Having a type system allows programmers to document their invariants, and have the machine check these invariants. Also, when someone wants to use a third-party library, having type definitions for the library makes it easier to understand how to call library methods. Last, JavaScript operators work on values of most types, and the conversion rules are very complex and hard to remember (for a fun take on this, see this [talk](https://www.destroyallsoftware.com/talks/wat)). A type system removes that complexity by simply forbidding most implicit conversions, e.g., the minus operator can only take numbers.
 
By proposing an optional type system instead of a sound gradual one, it becomes possible for developers who prefer untyped JS to largely ignore types. They can write in untyped JS and call into typed code without any changes to their code base.
	
For these reasons, we find that the cost/benefit ratio is well balanced.

## What’s the relationship between this and typed arrays?

TODO(goto): articulate this better.
NOTE(adamk): Not sure what this question means...are you worried people will confuse this use of "type" for the one in "typed arrays"? I'd be surprised by that (but would be interested if you've heard that feedback).

## Are transpilers still needed?

TODO(goto): articulate this better.

## Is TC39 the right venue?
 
We believe that the type system should be baked into the standard language and that TC39 is the right venue.
However, this proposal is primarily targeted towards a [typechecker](#terminology) used at [development-time](#terminology) rather than [production-time](#terminology) [interpreters](#terminology), and, in its current formation, there aren’t that many recurring representatives of developer tools (e.g. compilers, transpilers, minifiers, debuggers, profilers, etc).

Our hope is that TC39 would be open to introducing a venue/channel where authors of tools feel comfortable and interested to come and collaborate on the evolution of the typesystem.

@erights: from a specification perspective, to solidify the separation between the interpretation semantics and the typechecking semantics, it might be worth exploring describing them in separate docs, such as how JSON and i18n is handled.

## Network and parsing cost?
 
Adding extra syntax to the language that is meant to be ignored at runtime can lead to developers sending extra bytes on the wire that will be parsed but not used.

To mitigate that, one strategy would be to rely on minifiers/preprocessors/compilers to strip the metadata out prior to deployment. We think that a combination of (a) not using the optional type syntax, (b) paying the neglectable runtime cost for small apps that are already paying the cost with comments or (c) paying the transpiler/compiler/minifier cost offers a reasonable spectrum of solutions for developers.

## Backwards compatibility?
 
If a developer uses the type system, how do they deal with browsers that don’t support it yet (and will throw syntax errors)? Does that force one to use minifiers (i.e. as opposed to other JavaScript features where one can check for their existence on the fly)?
 
This seems like an analogous challenge with any proposal that changes the syntax of the language (e.g. async/await), so most/all of the trade-offs apply.

# Prior Art
 
This is by far not a new idea.
 
It appears in other programming languages, as polyfills and as previous proposals to JavaScript. Here are the most relevant things we have looked at (feel free to let us know if we are forgetting anything in particular).

## Other languages?
 
Python and Dart are most probably the closest analogies so we start with those. We go over a complete set of languages we found had related/interesting type systems at the end of this section.
 
### Python

From [PEP 0484](https://www.python.org/dev/peps/pep-0484/) and [PEP 482](https://www.python.org/dev/peps/pep-0482/).

* Python: [function annotations](http://legacy.python.org/dev/peps/pep-3107/) and [type hints](https://www.python.org/dev/peps/pep-0484/)
* Python’s [Function Annotations](http://legacy.python.org/dev/peps/pep-3107/) leaves semantics deliberately out
* Python’s [Type Hints](https://www.python.org/dev/peps/pep-0484/) standardizes semantics
* [Literature overview](https://www.python.org/dev/peps/pep-0482/) for Python’s type hints
* Python [@contract’s annotations](https://andreacensi.github.io/contracts/) for type systems

"While these annotations are available at runtime through the usual __annotations__ attribute, no type checking happens at runtime . Instead, the proposal assumes the existence of a separate off-line type checker which users can run over their source code voluntarily."

"The type system supports unions, generic types, and a special type named Any which is consistent with (i.e. assignable to and from) all types."

TODO(goto): give a better overview of PEP0482.
TODO(goto): chat with @collinwinter to gather ideas from python.

```python
>>> def greeting(name: str) -> str:
    return 'Hello ' + name
>>> greeting('Sam')
'Hello Sam'
>>> greeting.__annotations__
{'name': <class 'str'>, 'return': <class 'str'>}
```

No first-class syntax support for explicitly marking variables as being of a specific type is added by this PEP. To help with type inference in complex cases, a comment of the following format may be used:

```python
x = []  # type: List[Employee]
x, y, z = [], [], []  # type: List[int], List[int], List[str]
x, y, z = [], [], []  # type: (List[int], List[int], List[str])
a, b, *c = range(5)   # type: float, float, List[float]
x = [
   1,
   2,
]  # type: List[int]
```

By default generic types are considered invariant in all type variables, which means that values for variables annotated with types like List[Employee] must exactly match the type annotation -- no subclasses or superclasses of the type parameter (in this example Employee ) are allowed.

To facilitate the declaration of container types where covariant or contravariant type checking is acceptable, type variables accept keyword arguments covariant=True or contravariant=True .

### Dart

TODO(goto): write down about dart.

* [Optional Types in Dart](https://www.dartlang.org/articles/language/optional-types)
* [A Stronger Dart for Everyone](http://news.dartlang.org/2017/06/a-stronger-dart-for-everyone.html)

### Related languages

* [PHP’s gradual typing system](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration): type hinting
* Facebook’s [Hack](http://hacklang.org/) language, a typed variant of PHP.
* [Strongtalk](http://www.strongtalk.org/): optional type checking for smalltalk
* [Strongtalk: typechecking smalltalk in a production environment](http://www.bracha.org/oopsla93.pdf)
* [Ruby’s gradual typing system](http://blog.codeclimate.com/blog/2014/05/06/gradual-type-checking-for-ruby/)
* Java8’s [Type Annotations and Pluggable Type Systems](http://docs.oracle.com/javase/tutorial/java/annotations/type_annotations.html)
* [Type System as Macros](http://www.ccs.neu.edu/home/stchang/pubs/ckg-popl2017.pdf)
* Scheme Steering Committee [Position Statement](http://scheme-reports.org/2009/position-statement.html)
* [Lbstanza optional types](http://lbstanza.org/optional_typing.html)
* [N4j](https://numberfour.github.io/n4js/faq/comparison-typescript.html)
* [BabyJ](http://pubs.doc.ic.ac.uk/ObjectBasedToClassBased/ObjectBasedToClassBased.pdf) was a research exploration of optional types for JS. 
* [In defense of Soundiness: a Manifesto](http://dimvar.github.io/papers/soundiness-preprint.pdf)
* [Is Sound Gradual Typing dead?](http://www.ccs.neu.edu/racket/pubs/popl16-tfgnvf.pdf)

## TC39 discussions?
 
In TC39 this isn’t a new idea either. Here are the discussions we were able to find chronologically (feel free to send us links if you have any that we are missing here):
 
* 2002 [Javascript 2.0: Evolving a Language for Evolving Systems](http://www-archive.mozilla.org/js/language/evolvingJS.pdf)
* 2006 [Type parameters](http://web.archive.org/web/20160425220933/http://wiki.ecmascript.org/doku.php?id=proposals:type_parameters), [Type System](http://web.archive.org/web/20141214152853/http://wiki.ecmascript.org/doku.php?id=clarification:type_system) and [Structural Types and typing of initializers](http://web.archive.org/web/20150622061920/http://wiki.ecmascript.org/doku.php?id=proposals:structural_types_and_typing_of_initializers)
* 2011 [Dependent Types for Javascript](https://arxiv.org/abs/1112.4106) ([pdf](http://people.cs.uchicago.edu/~rchugh/static/papers/oopsla12-djs.pdf))
* 2011 [Guards](http://web.archive.org/web/20161123223114/http://wiki.ecmascript.org:80/doku.php?id=strawman:guards) and [Trademarks](http://web.archive.org/web/20141214075933/http://wiki.ecmascript.org/doku.php?id=strawman:trademarks)
* 2014 [TC39 Discussion on Types](https://github.com/rwaldron/tc39-notes/blob/master/es6/2014-09/sept-25.md#types): adding syntax without semantics.
* 2015 [ES8 gradual typing](https://esdiscuss.org/topic/es8-proposal-optional-static-typing) and [part2](https://esdiscuss.org/topic/optional-static-typing-part-2) and [ecmascript-types](https://github.com/sirisian/ecmascript-types).

# The Type System

## Strategy

* To pick the minimal amount of features that lead to a cohesive, coherent and usable type system (see [sequencing](#sequencing)).
* With the **benefit of hindsight**, to be a **strict subset** of production quality and battle-tested type systems (typescript, flow or closure), rather than a testbed for research (see [new ideas](#new-ideas)).
* A **good and materialized** type system is better than a **perfect and hypothetical** one.

## Sequencing?
 
Here are some of the features available in TypeScript/Flow/Closure that we chose to leave as future work. We don’t believe leaving any of these features out will corner ourselves into adding them later.
 
* What’s the the [maximally-minimal](http://wirfs-brock.com/allen/files/papers/standpats-asianplop2016.pdf) subset of features that will lead to a usable type system. Defer features with complex semantics (such as generics) or with dependencies in flight (e.g. typing public/private members) to a later version.
* Generics
* Enums
* Intersection types
* Public/private fields
* Function overloading
* Type Aliases
* Tuples
* Type guards
* Ambients
* Void
* Shorthand for optional types: TypeScript and Flow uses “?” preceding the variable name, whereas Closure uses “=” succeeding the typevariable name.

## New ideas?

Our [general strategy](#design-principles) is to pick a conservative subset of existing typesystems, rather than invent new things. We believe that [transpilers](#terminology) are a much more effective venue to innovate and experiment and that, with the test of time, features move from them to the standard language. We encourage you to work with the [transpilers](#terminology) and use them as a place to prototype your ideas and gather users before extending the [MVP](#terminology).

## Open Design Questions?

* Pervasive type inference vs. explicit types everywhere (big one)
* Disallowing implicit coercions 
* Union types in MVP or not
* Generics in MVP or not or special-cases only 
* Shorthands in MVP or not (includes: nullability/undefinedability ?, array [], maybe even function => as opposed to generics) 
* Most things in the subtyping rules? 
* Null vs. undefined handling, maybe

## This is more restrictive than JS. Is that on purpose?
 
For example one may feel very comfortable doing -x when one knows x is either a number or a string which I want to convert to a number; The alternative proposed here would require something like -Number(x).
 
That’s an accurate observation and that’s an accurate representation of the intention.
 
The three type systems made the choice to exclude some valid programs in the interest of finding useful warnings.
 
If we wanted to only warn for things that throw, then we would warn when calling a non-function, and when accessing properties on null/undefined, but not much else. Most JS operators work on all types, even though they are used with only a subset of types the vast majority of the time.
 
NOTE(domenic): there is a conflict between static analysis and dynamic analysis. Consider the example let x : number = true ? 0 : false; A runtime type system would have no problem with this, whereas what you're proposing would disallow it. This is probably worth highlighting at a higher level in the document: it seems like you are leaning toward a statically-checkable model, which there is precedent for for JS but I believe other optionally-typed languages went a different direction.

## Are you breaking the web?

Nope. See [are all existing programs correct?](#are-all-existing-programs-correct).

## Are all existing programs correct?

NOTE(erights): specifically, are they all still statically valid? and if not, how do you avoid #breakingtheweb?

[Interpreters](#terminology) remain unchanged, hence keeping all existing programs correct.

For the newly introduced [typechecker](#terminology) some (previously valid) programs may be (desirably) invalidated.

```javascript
var x = 'foo'; // OK
var y= x - 1; // OK
var z = 1 - true; // Error
// Typechecker Error, although Interpretation remains semantically and statically valid.
```

## What's the default for untyped programs?

All unannotated variable/parameter/etc is the same as annotating it with Any.

NOTE(erights): does all untyped code default to Any? We found in our experience with another language that it is useful sometime to use an unamed Type to be a better default in certain occasions / compilation contexts.


## Shorthands?

TODO(goto): for nullable types, optional types and arrays.

## Are varargs supported?
 
TODO(goto, domenic): are they?
 
## Are spread operators supported?
 
TODO(goto, domenic): are they?

## Generics

### Can you get away without generics?
 
We all agree that we want generics long term.
 
The choice to omit them now is just because of complexity. Specifying the type semantics will be very difficult, and we want to avoid as much difficulty as we can in the first version. We believe that the most significant hurdle for types in JS is to agree as a community on the kind of type system we want, and if we manage to get a type system in the standard, then progress on subsequent features will be easier.
 
From the experience with implementing type systems, we believe that generics are one of the most complex parts of the system, especially when combined with union types (so, Java generics are easier for instance).
 
Another very complex part to specify is flow sensitive typing. All three type systems have it, and we want it in the standard, but we chose to leave it out of the first version. 
 
If we leave out generics from v0 and add them in v1, backwards compatibility is possible because generics can always be instantiated with the any type, so Promise would be the same as Promise<any>.
 
Then again, as we say in other places, most aspects of this proposal are subject to change. So, I wouldn't be surprised if enough people want generics in v0 and we end up including them.

### Are Arrays, Promises and Iterables useful without generics?
 
And what about Thenable<>? Generator<>? Map<>/Set<>/WeakMap<>/WeakSet<>?

### Are you sure that with that strategy to special-case Arrays you can make it 100% backwards compatible?

NOTE(domenic): I'm familiar with at least C++ and C# as two languages that special-cased arrays before introducing generics and the result was a terrible mismatch carried forward into eternity. It's a pretty different situation, but it raises a warning alarm for me :).

NOTE(domenic): Basically C# arrays never implemented the generic collection hierarchy, so were incompatible with code that assumed generics. This led to advice that library authors write overloads, one for non-generic arrays and one for generic collections. And when library authors did not, authors needed to go through casts. Authors also needed to go through casts whenever they wanted to use language features that assumed a generic collection (e.g. LINQ). As I said, a pretty different situation than what's being done here :). Just a general area of warning bells.

### What are the types of access to arrays?
 
For example, in let a: Array[string];, does one need to define let b : string | undefined = a[0]; ? does that mean that the result of accessing an array can always lead to undefined? https://flow.org/en/docs/types/arrays/
 
NOTE(dimvar): This is a classic example of why unsound type systems are necessary for javascript, because bounds in array are always unchecked.
NOTE(dimvar): You can either make it sound by making the runtime check, or you can add the undefined everywhere (which is non ergonomical) and otherwise dependent types.

### What about if I do `= undefined`?
 
Trailing parameters with default values should be considered optional by the type checker.
 
NOTE(domenic): This seems subtle and hard to get my head around. (But not a real problem, just something tricky to work through.) E.g. if I declare this function: function a(value: number = 5) is that a "syntax error" since I didn't write number | undefined?


