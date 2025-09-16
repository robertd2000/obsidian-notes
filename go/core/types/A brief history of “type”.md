https://arcanesentiment.blogspot.com/2015/01/a-brief-history-of-type.html

The word “type” has a variety of meanings in programming languages, which are often a focus of confusion and contention. Here's a history of its use, focusing on particularly influential languages and papers.

### 1956: Fortran “modes”

The term “type” was apparently not yet established in 1956, because the Fortran manual speaks of integer and floating-point “modes” instead. It has something called “statement types”, but those are what are now called syntactic forms: assignment, conditional, `do`-loop, etc.

The 1963 [Fortran II manual](http://archive.computerhistory.org/resources/text/Fortran/102663119.05.01.acc.pdf) speaks of "two types of constants" (integer and floating-point), but this seems to be just the English word. When it talks about these types in more detail, it calls them “modes”, e.g. “arguments presented by the CALL statement must agree in number, order, mode, and array size with the corresponding arguments in the SUBROUTINE statement”. (Evidently the terms “formal” and “actual” parameters weren't established yet either.)

### 1958-63: Algol

Algol is one of the most influential languages in history. It introduced `if ... then ... else`, the `int n` declaration syntax, and semicolons. It also popularized the term “type”. The [Algol 58 report](http://www.softwarepreservation.org/projects/ALGOL/report/Algol58_preliminary_report_CACM.pdf) defines type declarations on variables in terms of the “type” and “class” of values:

> Type declarations serve to declare certain variables, or functions, to represent quantities of a given class, such as the class of integers or class of Boolean values. [...] Throughout the program, the variables, or functions named by the identifiers I, are constrained to refer only to quantities of the type indicated by the declarator.

The [Algol 60 report](http://www.masswerk.at/algol60/report.htm) is more consistent:

> The various “types” (`integer`, `real`, `Boolean`) basically denote properties of values. The types associated with syntactic units refer to the values of these units.

Note that types are explicitly a property of values, not variables or expressions. But does “basically” mean someone thought otherwise, or just that this isn't a formal definition?

### 1967: Strachey's Fundamental Concepts

Chris Strachey's [Fundamental Concepts in Programming Languages](http://www.itu.dk/courses/BPRD/E2009/fundamental-1967.pdf%E2%80%8E) was an influential set of lecture notes that established a bunch of common terms. It defines types thus:

> Most programming languages deal with more than one sort of object—for example with integers and floating point numbers and labels and procedures. We shall call each of these a different type and spend a little time examining the concept of type and trying to clarify it.

Strachey takes it for granted that types can be static or dynamic, and prefers static typing only for reasons of efficiency (which was, after all, of overwhelming importance in 1967):

> It is natural to ask whether type is an attribute of an L-value or of an R-value—of a location or of its content. The answer to this question turns out to be a matter of language design, and the choice affects the amount of work, which can be done when a program is compiled as opposed to that which must be postponed until it is run.

Strachey does not mention type theory, because no one had yet realized that it could be applied to programs. That changed in the next year.

### 1968: type theory

James Morris was the first to apply type theory to programming languages, in his 1968 [Lambda-calculus models of programming languages](http://dspace.mit.edu/bitstream/handle/1721.1/64850/23882173.pdf?sequence=1). “A system of types and type declarations is developed for the lambda-calculus and its semantic assumptions are identified. The system is shown to be adequate in the sense that it permits a preprocessor to check formulae prior to evaluation to prevent type errors.”

He begins by explaining what types are and why they matter, using the term in the usual programming-languages sense:

> In general, the type system of a programming language calls for a partitioning of the universe of values presumed for the language. Each subset of this partition is called a type.
> 
> From a purely formal viewpoint, types constitute something of a complication. One would feel freer with a system in which there was only one type of object. Certain subclasses of the universe may have distinctive properties, but that does not necessiate an _a priori_ classification into types. If types have no official status in a programming language, the user need not bother with declarations or type checking. To be sure, he must know what sorts of objects he is talking about, but it is unlikely that their critical properties can be summarized by a simple type system (e.g., prime numbers, ordered lists of numbers, ages, dates, etc.).
> 
> Nevertheless, there are good, pragmatic reasons for including a type system in the specifications of a language. The basic fact is that people _believe_ in types. A number is a different kind of thing from a pair of numbers; notwithstanding the fact that pairs can be represented by numbers. It is unlikely that we would be interested in the second component of 3 or the square root of < 2,5 >. Given such predispositions of human language users, it behooves the language designer to incorporate distinctions between types into his language. Doing so permits an implementer of the language to choose different representations for different types of objects, taking advantage of the limited contexts in which they will be used.
> 
> Even though a type system is presumably derived from the natural prejudices of a general user community, there is no guarantee that the tenets of the type system will be natural to individual programmers. Therefore it is important that the type restrictions be simple to explain and learn. Furthermore, it is helpful if the processors of the language detect and report on violations of the type restrictions in programs submitted to them. This activity is called type-checking.

Then he switches without explanation to taking about static checkers, e.g:

> We shall now introduce a type system which, in effect, singles out a decidable subset of those wfes that are safe; i.e., cannot given rise to ERRORs. This will disqualify certain wfes which do not, in fact, cause ERRORS and thus reduce the expressive power of the language.

So the confusion between programming-language and type-theory senses of the word began with the very first paper to use the latter.

### 1968: APL

APL-360 was the most popular dialect of APL. Its manual doesn't use the word “type”; it speaks of “representations” of numbers. But it considers these an implementation detail, not an important part of its semantics.

APL has a lot of unique terminology — monad and dyad for unary and binary operators, adverb and conjunction for high-order operators, and so on — so it's not surprising that it has its own word for types too.

### 1970: Pascal

Wirth's 1970 definition of Pascal is, as usual, plain-spoken: “The type of a variable essentially defines the set of values that may be assumed by that variable.” (But there's that “essentially”, like Algol's “basically”.)

### 1970-73: Lisp belatedly adopts the term

Like Fortran, early Lisps used the word “type”, but only in its ordinary English sense, never as a technical term. AIM-19, from 1960 or 1961, speaks of “each type of LISP quantity”, but doesn't use “type” unqualified. Similarly, the 1962 [Lisp 1.5 Manual](http://bitsavers.org/pdf/mit/rle_lisp/LISP_I_Programmers_Manual_Mar60.pdf) uses the word for various purposes, but not as an unqualified term for datatypes. The most common use is for function types (`subr` vs. `fsubr`); there are “types of variables” (normal, `special`, `common`), but datatypes were not, apparently, considered important enough to talk about. They might not have even been seen as a single concept — there are awkward phrases like “bits in the tag which specify that it is a number and what type it is”, which would be simpler with a concept of datatypes.

This changed in the early 1970s. The 1967 [AIM-116a](ftp://publications.ai.mit.edu/ai-publications/pdf/AIM-116a.pdf) and 1970 [AIM-190](ftp://publications.ai.mit.edu/ai-publications/pdf/AIM-190.pdf) still don't use “type”, but the 1973 [Maclisp manual](http://www.saildart.org/MACLSP.DBA%5BUP,DOC%5D1) and 1974 [Moonual](http://www.softwarepreservation.org/projects/LISP/MIT/Moon-MACLISP_Reference_Manual-Apr_08_1974.pdf) do, and it consistently means “data type”. Most tellingly, they have `typep`, so the term was solidly ensconced in the name of a fundamental operator.

### 1973: Types are not (just) sets

By 1973, the definition of types as sets of values was standard enough that James Morris wrote a paper arguing against it: “Types are not sets”. Well, not _just_ sets. He was talking about static typechecking, and argued that checking for abstraction-safety is an important use of static typechecking. The abstract explains:

> The title is not a statement of fact, of course, but an opinion about how language designers should think about types. There has been a natural tendency to look to mathematics for a consistent, precise notion of what types are. The point of view there is extensional: a type is a subset of the universe of values. While this approach may have served its purpose quite adequately in mathematics, defining programming language types in this way ignores some vital ideas. Some interesting developments following the extensional approach are the ALGOL-68 type system, Scott's theory, and Reynolds' system. While each of these lend valuable insight to programming languages, I feel they miss an important aspect of types. Rather than worry about what types are I shall focus on the role of type checking. Type checking seems to serve two distinct purposes: authentication and secrecy. Both are useful when a programmer undertakes to implement a class of abstract objects to be used by many other programmers. He usually proceeds by choosing a representation for the objects in terms of other objects and then writes the required operations to manipulate them.

### 1977: ML and modern static typing

ML [acquired its type system in about 1975](http://arcanesentiment.blogspot.com/2012/05/when-was-ml-invented.html) and was published in 1977. Until this point, the application of type theory to programming languages had been theoretical, and therefore had little influence. ML made it practical, which has probably contributed a lot to the terminological confusion.

ML's theoretical support (along with the misleading slogan “well-typed expressions do not go wrong”) came out in the 1978 paper [A Theory of Type Polymorphism in Programming](http://geofft.mit.edu/p/milner-type-poly.pdf), which despite being about type theory, speaks of types containing values:

> Some values have many types, and some have no type at all. In fact “wrong” has no type. But if a functional value has a type, then as long as it is applied to the right kind (type) of argument it will produce the right kind (type) of result—which cannot be “wrong”!
> 
> Now we wish to be able to show that—roughly speaking—an Exp expression evaluates (in an appropriate environment) to a value which has a type, and so cannot be wrong. In fact, we can give a sufficient syntactic condition that an expression has this robust quality; the condition is just that the expression has a “well-typing” with respect to the environment, which means that we can assign types to it and all its subexpressions in a way which satisfies certain laws.

### The short version

So here's the very brief history of “type” in programming languages:

1. It wasn't used at all until 1958.
2. Types as sets of values: Algol-58.
3. The type-theory sense: Morris 1968.

These may not be the earliest uses. I got most of the old manuals from [Paul McJones' collection](http://www.softwarepreservation.org/projects/lang), which is a good place to look for more. I welcome antedatings.

I'm also curious about the term “datatype”, which might plausibly be ancestral to “type”. I could find no uses of it older than “type”, but I may be looking in the wrong field. Statistical data processing is much older than computing, and has dealt with datatypes for a long time. Might the terms “datatype” and “type” have originated there?

Update August 2015: Jamie Andrews said much the same [seven months earlier](http://lists.seas.upenn.edu/pipermail/types-list/2014/001781.html).

Update June 2017: In [HN comments](https://news.ycombinator.com/item?id=14402778), dvt found [“datatype” in 1945, in Plankalkül](http://arcanesentiment.blogspot.com/2017/06/antedating-datatype-all-way-to.html).