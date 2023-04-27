+++
title = "(WIP) Types: The Foundation and Keystone of Programming"
date = 2023-04-05
draft = false

[taxonomies]
categories = ["Types"]
tags = ["Intro"]

[extra]
toc = true
keywords = "Types, Syntax, Semantics, Logic, Math, Foundation, Type Theory, Programming, Language"
+++

# Preface

This is some of my meandering thoughts about **types**, which I'll argue represent the formal ways in which we can reason about text <sup>[1]</sup> as a program. It's a semi-philosophical look at some of the fundamental sorts of reasoning that computer scientists do. I don’t expect reading this will make you a better programmer, but it may give you a fresh perspective when approaching types and I hope it's a fun read regardless.

If most of your experience with programming has by and large been with dynamically typed languages (Like Python or JavaScript) or languages with relatively simple type systems (like C++ or Java), and you're looking to expand your programming repertoire with one of the statically typed functional languages then I hope the {%emph()%}10000 foot view{%end%} 

This is an attempt to write out my understanding of the topic and should be taken as such. I am not an expert on types and am likely to provide an incomplete or perhaps even entirely wrong understanding of some of these concepts. 

The plan is to update this as my understanding evolves.

----

<sup>[1]</sup> Any data ~~structure~~ capable of encoding the structure of a program

# What is a Type?

When you open a Java Class file and see:

```Java
int a = 5;
```

or a Rust file and see:

```Rust
let a: i32 = 5;
```

`int` and `i32` are types and in this case and both denote a {% emph() %}32-bit signed two's complement integer{% end %}. The variable `a` is exactly one of `int`'s 2<sup>32</sup> (or 4294967296) possible inhabitants. There's some loose notion under which `a` denotes an element of `int` as though `int` were a set. If I had written `int a = 5.0,` I would have received a compilation error saying the expression is not well typed. The error would say something to the effect that a floating point number cannot be assigned to a variable meant to hold an integer.

At their core, types seem like a very simple, but frustratingly abstract thing. They have a somewhat liminal nature, existing in many forms directly in syntax — where rules inductively define well typed syntax — and also in the meta-language — where we leave it as an exercise to the reader to understand why some expressible syntax is not necessarily meaningful. 

That's rather vague; in programming languages types seem to be restrictions<sup>[1]</sup> on the symbolic construction of a program. For example, this can be seen as restrictions on which values can be assigned to a variable, passed to a function, appended to a list, and so on.

----

<sup>[1]</sup> Talking about types as *restrictions* on a language is perhaps an aesthetically poor choice.

- **First:** Dynamically typed languages have a few tricks that make it appear at first glance like they don't do this. They tend to use hidden control flow to look a bit like a language without a type system (We'll discuss dynamic typing further when it's relevant).
  
- **Second:** A related topic that we're not going to cover at length here is that the types in a language's type system can double as a *domain specific language*. This means that defining types not only encodes machine verifiable invariants, it may also specify computation.<br><br>
  Languages with rich type systems tend to come equipped with tools by which the compiler can generate incredible amounts of boilerplate code on behalf of the programmer. This means they tend to feel much more expressive in many cases. Serialization, deserialization, methods, algorithms, integration, testing, code reuse, and more can sometimes be automated based on types.<br><br>
  Some styles of design/API, like function overloading (letting the compiler select an implementation based on types), or dynamic dispatch (say: letting the compiler thread a virtual function table through part of a program) is only possible with an appropriate type system.

## Motivating the use of types

Lets jump into a simple (although contrived) example using a less expressive type system to demonstrate how this might be helpful. Here's a simple function you might find in the opening chapters of an *Introduction to Python* book:

```Python
def factorial(n):
  if n == 0:
    return 1
  else:
    return n * factorial(n-1)
```

If you pass this function a negative number, it will never hit the base-case of `n == 0` and will instead recurses endlessly, hanging the program and burning CPU cycles until something kills the process. In truth, Python has a recursion limit which it will eventually reach in this case, though if you re-write this function using a while loop instead of recursion you lose even that extra bit of safety.

Consider a relatively large, long-lived program. Say a web-server responding to queries. It's responding to millions of requests per day, but unfortunately, roughly once per month the server crashes when `factorial` is called. You deduce that there is some relatively rare sequence of events that results in `factorial` being called with some value for which it isn't well behaved. You're on a time crunch and you know your server is robust against throwing errors so you write a bit of code to solve the problem:

```Python
def factorial(n):
  if n < 0:
      raise "factorial only accepts numbers greater than zero :)"
  else:
      return __factorial(n)
      
def __factorial(n):
  if n == 0:
    return 1
  else:
    return n * factorial(n-1)
```

Now you've partially solved the issue. Once a month or so, the server surprises a client by returning some error code, but at least the server doesn't crash. Only the issue regresses when a new feature requires the following class. Almost immediately, a junior dev copies and pastes some code and accidentally passes an instance of `Color` to `factorial`.

```Python
class Color:
    def __init__(self, r,g,b):
        self.r = r
        self.g = g
        self.b = b
        
    def __add__(self, v):
        if isinstance(v, int):
            return Color(
                self.r + v,
                self.g + v,
                self.b + v
            )
        elif isinstance(v, Color):
            return Color(
                (self.r + v.r) % 250,
                (self.g + v.g) % 250,
                (self.b + v.b) % 250,
            )
        else:
            return self
            
    def __sub__(self, v):
        if isinstance(v, int):
            return self.__add__(-v)
        elif isinstance(v, Color):
            return self.__add__(Color(-v.r,-v.g,-v.b))
        else:
            return self
        
    def __mul__(self, v):
        if isinstance(v, int):
            return Color(
                self.r * v,
                self.g * v,
                self.b * v
            )
        elif isinstance(v, Color):
            return Color(
                (self.r * v.r) % 250,
                (self.g * v.g) % 250,
                (self.b * v.b) % 250,
            )
        else:
            return self
        
    def __eq__(self, v):
        try:
            return (self.r == v.r 
                and self.g == v.g
                and self.b == v.b)
        except:
            return False
    
    def __lt__(self, v):
        try:
            return (self.r < v.r 
                and self.g < v.g
                and self.b < v.b)
        except:
            return False
```

Because `Color` is defined "just so", passing it to factorial will crash the server again. **Bummer**. You could solve this with some more code, though because factorial is used on the hot-path, your server's performance is suffering. Instead, you take the time to hunt down the known bugs. YThen, because response time matters, you revert back to the more performant version of factorial. Then you write a comment explaining the function's precondition.

```Python
# !!!IMPORTANT!!! If you're reviewing code that calls this function, double and 
# triple check that it is being called with a positive integer. 
def factorial(n):
  if n == 0:
    return 1
  else:
    return n * factorial(n-1)
```

With a hundred developers working on the code base, is it the case that the best you can do is cross your fingers and hope everybody understands how to call this function? Well, not really. It turns out that a type system can solve this problem statically without any performance penalty (for the response time of your server). Here's how you might port this function to rust:

```Rust
fn factorial(n: u32) -> u32 {
    if n == 0 {
        1
    } else {
        n * factorial(n - 1)
    }
}
```

This signature (`fn:(u32) -> u32`) effectively says "calling factorial with a positive integer will return a positive integer, calling it with any other value is undefined behaviour and should be avoided." Alone, this isn't any better than a comment, fortunately Rust comes paired with a type checker that is able to check every invocation of `factorial` in the code for your server and confidently assure you that none of the hundred developers working on the code base has accidentally invoked `factorial` with any other {%emph()%}type{%end%} of value. This incurs a penalty during compilation, but it doesn't affect the run-time of your program and it ensures that certain bugs never appear in the compiled artifact.

For `factorial`, this is only a small win. After all the name seems descriptive enough that it feels quite unlikely a developer would misuse it. While I agree that this is a contrived example, I hope this blog will convince you that it takes relatively few building blocks to form the foundation for surprisingly robust type-level systems. There are design patterns and software architectures that can be encoded (and enforced) at least in part in the language itself using its type system. 

For instance, Rust's type system is expressive enough to encode an SQL schema and ensure that any statically written query does not compile if it doesn't meet the schema, it can also automaticaly generate code that checks dynamic queries at runtime. You can generate these types automatically at compile time by querying your SQL server for the most up to date schema. Those types can be available across the entire program meaning that information about whether some value has been checked against the schema for correctness can be encoded in it's type and passed across isolated subsystems without the fear that it hasn't been sanitised and without performing the sort of "shot-gun validation" in which values are checked everywhere in the hopes that every error is caught somewhere.

Writing code knowing that future developers (including yourself) will be forced to meet its pre-conditions is generally liberating. You can push the responsibility for type safety backwards onto callers. Conversly, you can perform some computation to parse a less percise representation into a more percise one knowing that developers writting downstream code will never need to audit the program to be sure a value meets certain criteria.

> In other words, write functions on the data representation you wish you had, not the data representation you are given. The design process then becomes an exercise in bridging the gap, often by working from both ends until they meet somewhere in the middle.
> - Alexis King — [Parse, don’t validate (link)](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/)

This style of software design is certainly possible in an assembly language, but as the size of the codebase and the number of developers increases, such designs are hard to maintain. The problem is that every time you update (for new feature or bug fix) your codebase, 

While jumping into a Haskell codebase and implementing some new feature is often daunting and difficult, it is also generally more difficult to do wrong since the compiler can perform exhaustiveness checks to ensure that your functions are total, that errors are handled, and that any new code type checks against everything else in the codebase.

In languages without type *systems*, types still appear in the margins. Programmers writing assembly still work with basic types like:

- {% emph() %}boolean{% end %}, 
- {% emph() %}string{% end %}, 
- {% emph() %}integer{% end %}, 
- {% emph() %}floating point number{% end %},  
and so on.


They also work with more abstract types like: 
- {% emph() %}first and last name{% end %}, 
- {% emph() %}recoverable errored socket connection{% end %}, 
- {% emph() %}opened file handle{% end %}, 
- {% emph() %}any data type that allows a structure-preserving mapping between pointers to heap allocated values paired with a function pointer that can compute such a mapping{% end %},  
and so on.

These types appear directly and/or obliquely in comments, notes, and design documents.

Tracking types throughout your program is tedious and error-prone, but can fortunately be automated. Having a type system (static, dynamic, or both) is standard in programming languages because of how pervasive and foundational types are when wanting to correctly instruct a computer.

# The Naive View of Types in Computer Science

> [A] computer is unable to discriminate between for example a memory address and an instruction code, or between a character, an integer, or a floating-point number, because it makes no intrinsic distinction between any of the possible values that a sequence of bits might *mean*
>
> [...]
>
> Assigning a data type, termed typing, gives *meaning* to a sequence of bits such as a value in memory
> - [Wikipedia link](https://en.wikipedia.org/wiki/Type_system)

Consider the following sequence of bits:

```
10000001
```

If asked, a computer scientist might have a few thoughts about what this represents as a value:

 - If that's an unsigned 8bit integer, then it could represent a `129`.
 - It also could represent any constant transformation of an 8bit integer:
   - Absolute negative value: `-129`
   - Thousands: `129,000`
   - Even numbers: `258`
 - If that's a signed 8bit integer then it could represent a `-1`, or if it is in 2s complement then `-127`.
 - If the bit that carries the sign isn't the standard one, there's a whole other set of numbers this could represent.
 - It could be an array of two unsigned 4bit integers: `[8,1]`
 - The value may not be an integer at all. Getting more outlandish, it might be a bit-map for items in an adventurer's backpack. In which case this might encode two specific key items (item-id `0` and `7`) out of eight key items an adventurer will need to find during their quest! 
 - This value could be a memory address on a **very** small embedded system, or perhaps the system pads this with zeros to interpret it as a memory address.
 - This could be a machine instruction code of some sort.
 - This could be a path through a binary tree.
 - The above considerations also depend on the "endianness" of the memory layout as well.

You can see the problem. You cannot tell the type of this value by inspecting its bits. Giving those bits a type is the process by which we imbue them with meaning. It doesn't matter that assembly has no means to annotate types and no automated way to check them, any programmer using assembly must do that work manually — they can not reasonably choose to avoid it. For example: it's never desirable to multiply two memory addresses together, so an assembly programmer does their best to avoid doing so. Failing in this task isn't a type error within the language, but it is still a type error conceptually and will result in an incorrect program.

Programmers need to reason about the types of the data they are working with in order to ensure that their programs are correct. Note that programs are themselves data whose 0s and 1s are imbued with meaning and which developers must be able to reason about.

# Typing Judgments are an Exercise in Abstraction

So far I've quoted and discussed the view that:

> Assigning a data type, termed typing, gives *meaning* to a sequence of bits such as a value in memory.

While that is a way to motivate the use of types, this is not typically how types act while programming. The easiest way to understand this is to consider how often a developer even sees the sequence of bits that represent the values they use. In general, programmers consider even the most fundamental types to have an existential representation. A well behaved representation exists (because it must), but any other well behaved representation would do just as well. Many Java programmers have never checked (and it has never mattered) whether their `int` is represented in 2's compliment or not.

The type system forms an abstraction boundary so that so long as the API of an `int` doesn't change, the representation is largely irrelevant. Consider that this type of abstraction can be and often is applied to programs. For example: when you query a database, you typically care that the correct result is returned quickly and not **how** it is returned quickly. 

There is a connection to reasoning about the properties of a program separately from the program’s implementation that translates into reasoning about the properties of a type separate from their representation (In formal verification circles, they're the same style of reasoning. Full stop).

What sorts of meaning a type system can express varies wildly from language to language. For example, in C just about everything is a number and you can accidentally multiply `North` by `Blue` since the type system can only understand their integer encoding. In Haskell, `North` may have the same run-time representation as an integer, but it has a different type. Once again, this just reinforces that a complete lack of a type system or an impoverished type system simply means that any work done by one developer to reason about types is either lost or must be passed on through some other means.

> The fundamental problem addressed by a type system is to ensure that programs have meaning. The fundamental problem caused by a type system is that meaningful programs may not have meanings ascribed to them. The quest for richer type systems results from this tension.

### Shoving type information into the run-time

**An aside for dynamic language enthusiasts**.

You can put type information into the run-time of a program. 

The best known (and universally applauded) type that does this is the signed integer. Statically, a signed integer is a sum type. One bit is reserved to tell the system how to interpret the other bits. For example: if the tag bit is a `1`, the other bits represent the absolute 'value plus `1`' of a negative number (The 'plus `1`' means there's no way to represent `-0` as distinct from `+0`). If the tag bit is a zero, the other bits represent a positive number. Like any type, how this is done is down to convention. As long as disparate parts of the system/program agree on which bit is the front-matter for a signed integer, there is no further intrinsic meaning to the choice.

This is a type that requires an invariant not explicit to the construction of the type being upheld. That is, that the front-matter bit faithfully describes what follows. If your system ever mismanages the front-matter, then `7 - 10` becomes `3` instead of `-3` (or `2147483645` instead of `-3` given 2s compliment). There are strict rules for how the system is allowed to interact with such a type in order to maintain this invariant.

Being the tail part of a sum type changes the way you reason about a type, but it still meets everything else we discussed about types; both in giving values meaning and in abstracting representation.

This is a pattern that can be used in a broader context and even embedded directly into a language. A programmer can statically make a choice to allocate some extra front-matter to a value which can then be used to discern how to interpret the rest of the value. Any language with a tagged union, variant, variant record, choice type, discriminated union, disjoint union, sum type, coproduct, etc is giving developers the ability to create such types.

Dynamic languages like JavaScript take this idea to an extreme and create a massive discriminated union of all their types. Statically, their values can be introspected into any known run-time type. There is a trade-off made here. Every value must be bigger due to this front-matter and the run-time must consistently and repeatedly check the front-matter whenever a typed operation is performed (Modern interpreters/compilers for dynamic languages can use static analysis to infer some static typing and check types a bit less incessantly). Such languages tend to rely on hidden/implicit control flow — for example, throwing an error — to handle all the extra possible run-time errors. From a static perspective, this is a return to untyped languages like assembly. Again, developers must either develop a typed understanding of their program or be forever baffled by run-time type errors.

On the other hand, there are benefits to this approach as well. When programming with a dynamic language your reasoning can be a bit fast and loose since type errors are still caught at run-time. Printing a value can implicitly inspect the discriminated union in which the language encodes all its run-time values. Developers no longer have the issue where there is no way to tell a value's type at run-time (They can blindly assume some standard front-matter). This enables a style of programming where developers can stop reasoning about the possible states of their program and reason only about a state they've seen. While error prone, it's certainly a fast way to accomplish smaller tasks and explains why most scripting languages are dynamically typed.

It's an important point to reiterate, however, that types can be seen as a conceptual construct that exists independently of any particular type system. Not all programming is done using types. For instance: programming can be done through copying and pasting code from online forums and adjusting a few parts with very little understanding of the program being created. In theory, you can generate random text for a compiler until you randomly compile a program that performs some task as desired. However, generally developers program using their reason to understand and solve problems. Pretty much all such reasoning can be put under the purview of type theory. 

That's a bold claim and while I can't fully substantiate it, I hope to describe why the notion isn't too wild. 

# Foundational Mathematics

Types aren't **just** a computer science concept. In the broader context of math, they can be used to restrict which statements a mathematician spends time pondering. In set theory, for instance `π ∈ cos` is a well-formed mathematical statement whose truth value is hopefully false but doesn't materially matter (It's a silly proposition, or in other words it's not a well typed statement).

Propositions like `3 ∈ π` make structuralists uncomfortable. If you ask a structuralist whether this proposition is true, they’ll immediately say "that makes no sense. The answer to such a question has nothing to do with 3 or π as things in themselves, or even as things in ℝ — it’s a question about a particular encoding in a particular universe."

A less intuitive but more direct example comes from contrasting Von Neuman's and Zermello's constructions for the Natural Numbers (ℕ — Numbers from 0 onward, you can call them the whole numbers if you prefer). In set theory, `0 ∈ 1` is a well formed mathematical statement which happens to be true under Von Neuman's ℕ and false under Zermello's ℕ. As I understand it, Set Theorists don't consider this a problem because they stick to the properties of a construction of ℕ only insofar as they're equal (up to isomorphism) with any other possible construction of ℕ. It turns out that propositions like `0 ∈ 1` are not.

The same issue resurfaces over and over again. For example:

> there are two obvious constructions of ℝ, as a set of equivalence classes of Cauchy Sequences of rationals, or as the set of Dedekind cuts of rational numbers. Both give definite truth values to “silly” questions, but these answers are not properties of 3, π or ℝ, they’re properties of the encoding, and passing to a different encoding changes these properties, even though the essential properties of ℝ are preserved.
>
> The structuralists’ argument is that we shouldn’t be able to ask questions which aren’t structurally invariant.
> - Professor Michael Shulman ([link](https://golem.ph.utexas.edu/category/2013/01/from_set_theory_to_type_theory.html))

Just like a structuralist, a type theorist doesn't need to abstract such statements away, the equivalent statement in Type Theory can't be meaningfully constructed. There's a sense in which Type Theory for math and Type Systems for computer science are ways to reify notions like this from the meta-theory into the theory itself. It restricts one from constructing statements like `3 ∈ π`, or `5 + Cat()` because if expressed correctly, such statements will not be well typed. 

Types as they're understood in type theories are well documented, talked, and blogged about. I don't understand them well enough to do them justice, nor are they really what this article wants to talk about. I'll attempt to give it a very superficial treatment here because I think it frames the discourse about types I aim to have below. I don't intend to talk about foundational mathematics, but I do intend to develop the intuition that the sort of thinking that a developer does while they program is generally structured well enough that it can be described/annotated/formalized/etc.

More specifically, I want to discuss why I think programmers/developers use type-theoretic reasoning even if their language doesn't provide the means to annotate or check such reasoning for correctness. The idea that underpins much of this thinking is the Curry–Howard correspondence which shows a deep, reoccurring, and outwardly mysterious correspondence between logic and computing. 

> Propositions as Types is a notion with depth. It describes a correspondence between a given logic and a given programming
language. At the surface, it says that for each proposition in the logic there is a corresponding type in the programming language and vice versa. Thus we have:
>
>        *propositions as types.*
> 
> It goes deeper, in that for each proof of a given proposition, there is a program of the corresponding type—and vice versa. Thus we also have:
>
>        *proofs as programs.* 
> 
> And it goes deeper still, in that for each way to simplify a proof there is a corresponding way to evaluate a program—and vice versa. Thus we further have: 
> 
>        *simplification of proofs as evaluation of programs.*
> 
> Hence, we have not merely a shallow bijection between propositions and types, but a true isomorphism preserving the deep structure of proofs and programs, simplification and evaluation.
>
> Propositions as Types is a notion with breadth. It applies to a range of logics including propositional, predicate, second-order, intuitionistic, classical, modal, and linear. It underpins the foundations of functional programming, explaining features including functions, records, variants, parametric polymorphism, data abstraction, continuations, linear types, and session types. It has inspired automated proof assistants and programming languages.
> - Professor Philip Wadler ([link](http://www.cs.bc.edu/~muller/teaching/lc/WadlerPropositionsAsTypes.pdf))    
> (It's outside the scope of this article, but the paper linked above has a nice example comparing a proof and a program. Showing how — at least in one case — simplification of a proof acts just like evaluating a program.)

The most expressive type systems are found in languages like Idris, Agda, Coq, or Lean which were built to realize or take advantage of the Curry–Howard correspondence. When using a language like this, any invariant you might be able to express about a value can be expressed in its type.

Dependent type theories like Martin-Löf/Homotopy/Univalent Type Theory fit in the same abstract layer of mathematics as predicate logic + set theory. They're meant to provide a foundational framework from which the rest of math can be described. 

> The language we use in [type theory] is a bit different from that of set theory, but at the level of informal mathematics, the differences really amount to nothing more than changes in terminology and notation. Instead of “structural-sets” we speak of types. Sometimes we call their elements terms. And the quantifier-bounding membership is written as a:Z rather than a∈Z (here Z, of course, is a type — that is, a structural-set — while a is being given as an element of it). 
> - Professor Michael Shulman ([link](https://golem.ph.utexas.edu/category/2013/01/from_set_theory_to_type_theory.html))

It shouldn’t be surprising that a foundational math theory like type theory has something to say about computer science types. After all, it is designed to have something to say about every field of mathematics from algebra to number theory to statistics. 

On the other hand, a practising statistician has long since abstracted away from sets or types. They encode their problems and express their solutions in terms of abstractions (for example ℕ: The natural numbers). Computer scientists formalizing their type systems or discussing computational models tend to use type theory a bit more directly.

{% aside() %}
### Day to Day Logic

Let’s say I need to decide how many pairs of socks to take on vacation. I’m not content to just use a gut-feeling, so I think about it for a time and after any number of considerations I then decide to take 3 pairs of socks. 

Afterwards, I’m asked to formalize my reasoning using first-order predicate logic (FOL) without changing my reasoning. I could probably rephrase my reasoning in FOL terms. Stating my assumptions and constructing my reasons from there. On the other hand, in attempting to formalize it, I may realize my reasoning was fallacious and that 2 pairs of socks would be an easier to defend choice.

**Here’s a question:** What is the relationship between my reasoning and FOL? Are people doing FOL in their day-to-day lives when buying groceries or deciding the shortest route to the dog park? What of the idea that people can reason erroneously or perhaps neglect to invoke their reason at all?

This interplay is at work with Types, Developers, and Type Systems. Typically, it doesn't matter whether logic **describes** a style of thinking or a style of thinking **is** logical. If some part of what is said brushes your intuition incorrectly, see if re-framing it this way re-aligns this discussion for you.
{% end %}

# The Pervasiveness of Type Systems

There's a very compelling case to be made that type systems are given a different treatment depending on where they're used. C++'s Templating and TypeScript both use duck typing while Haskell has a nominal type system heavily based on Hindley–Milner type inference. Java and C hardly use inference at all and ask the developer to label their types in even the most obvious cases. Python shoves most of it's types into the run-time. These all seem like very different things and it may well be that any insight gained by understanding what unifies these type systems has a fleeting impact on the development of any given software system.

There's a lens through which this is a healthy way to look at things. When you learn a new language, the types are some part of the syntax and semantics. We don't typically look at common elements of syntax between languages and see any deeper meaning. Certainly the fact that so many languages use `{` and `}` is an artifact of history and doesn't offer some fundamental insight into the nature of programming languages.

Perhaps it **should** be surprising that a language’s ad-hoc system for labelling values with types is best described/expressed with mathematics’ foundational building blocks. After all, the Types as seen in dependent type theory feel far removed from the {% emph() %}int{% end %}, {% emph() %}long{% end %}, {% emph() %}float{% end %}, {% emph() %}double{% end %}, {% emph() %}Object{% end %}, or {% emph() %}SimpleBeanFactoryAwareAspectInstanceFactory{% end %} as seen in Java. Perhaps it should also be surprising that all these seemingly different systems (so often developed entirely independently of one another) should end up being related in curious and interesting ways.

**Here is my contention:** Insofar as they help developers, less expressive type systems can be seen as carving out little chunks of a grander underlying theory while leaving developers to reason the rest out on their own. More expressive type systems can capture or encode more abstract reasoning. Regardless of how directly useful such lines of thinking are, it's enriching to think along these lines. 

I’m reminded of how often I’ve seen and heard developers express that programming isn’t mathematics so much as it’s "*just logic*". The idea is that the exercise you’re engaged in while writing a program is much easier than, for example grade 11 Calculus. Instead, I suspect that — especially for shorter or simpler programs — the sorts of reasoning you invoke are much more foundational and much less abstract than some of the Math you’ve already learned. As different as it looks, a simple Agda program (for example, one that sorts a list) often just makes explicit the sort of reasoning a Python developer may implicitly perform while writing code.

Across all the various general purpose languages, the logical building blocks by which you program don’t really change too much. Pretty much every language with a type system is using types to solve the same problem. Whether they realize it or not, they're attempting to embed some form of type-like abstract reasoning in their language (so that it can be automatically checked). They do this (to varying degrees of success) to catch errors or improve productivity by localizing reasoning.

Languages with dynamic type systems do this with with a run-time cost and try to ensure type errors appear as errors rather than just strange behaviour. It's often possible to subvert the type system, allowing a program to elicit some undesired behaviour. 

# About Unsound Type Systems

Some languages have issues with their type system’s soundness. Java’s arrays are famous for the ease with which you can fool the type system. The following compiles just fine even though the type for `first` is wrong. 

```Java
Cat[] meows = {new Cat(10), new Cat(18)};
Animal[] lovable = meows;
lovable[0] = new Dog(13.33);
Cat first = meows[0];
```

Before Java introduced generics and a proper system for variance in data structures, this made some pretty common design patterns considerably more ergonomic at the cost of type system soundness. Java's Generics are one of many examples across many languages of a less expressive/more ad-hoc type system adopting useful parts of more expressive, more formal type systems.

A Java developer might say, "don't do this! Use the generic collections instead since they wouldn't allow this to compile, or be sure never to read from `meows`  and read only from `lovable` from now on". In other words, developers are still expected to reason about the correctness of their types (Passing the type checker isn't always enough). The thinking here seems to be that Java's type system isn't used for formal verification, but instead the type system is meant to be as helpful as is reasonable, catching most type errors automatically and helping refactor code in some cases. If used correctly it can now safely encode design patterns that once to require some extra care, but you need to know to do so.

In an even more extreme case, check out TypeScript's none-goals ([link](https://github.com/Microsoft/TypeScript/wiki/TypeScript-Design-Goals#non-goals)): number 3 says that TypeScript is not aiming to create a sound or "provably correct" type system.

**A question:** to what degree it is correct or even useful to think about types as they appear in C, C++, JavaScript, Java, Python, etc as being the same sort of mathematical thing as types as they appear in Agda, etc. Earnestly, I don’t really know the answer.

It seems to me that this is the state of affairs regardless of type system soundness. Short of full state space exploration or formal verification, even sound type systems may still have developers reasoning about their programs in a way that a more expressive type system could encode. You can think of an assembly languages' type system as being sound (*Everything is a series of bits! It can't be fooled!*), but of course before doing anything a developer will immediately layer their own more expressive types on top of those (They simply lose any ability to check them in an automated way).

**Another question:** Which is preferable?  

  1. The false sense of safety of an expressive, but ultimately flawed type system.
  2. The lack of safety of an impoverished type system.

Considering that a type system takes time both to learn and to use, it's unlikely massive software systems will get written in languages like Agda. The burden to hire developers and the extra effort required to use the type system may create an insurmountable problem for now. However, there's some hope, languages (like Haskell, Rust, etc) are experiments in what can be done to keep the type system expressive, sound, as well as productive.