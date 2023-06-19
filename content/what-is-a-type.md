+++
title = "What's a Type?"
date = 2023-06-09
draft = false

[taxonomies]
categories = ["Types"]
tags = ["Intro"]

[extra]
toc = true
keywords = "Types, Syntax, Semantics, Logic, Math, Foundation, Type Theory, Programming, Language"
+++

This is the first in a series of loosely related posts exploring Types, Type Theory, Type Systems, and some related content from the point of view of a computer scientist. The general idea is to provide a {%emph()%}10000 foot view{%end%} of what I've been learning on the topic that may inspire me (or others) to remember and learn more. 

This article takes a while to feel like it's about computer science, but it is! Really!

# Type Systems share some Similarities with Proof Systems

While tackling this topic, I'm hoping that you have some (perhaps partially forgotten) introductory-level knowledge of formal logic. I first learned some propositional logic in my Grade 11 Discrete Mathematics course, then again in undergraduate philosophy. I'm not going to lean on it in any in-depth way in this blog, but its a nice building block for understanding type systems and in turn, types.

While I don't want to turn this is an introduction to propositional logic, lets take a quick moment to refresh our memories on the two ways you can prove that `A ∧ B` entails `B ∧ A` in propositional logic.

### The Semantic Proof

The first way you can prove that that `A ∧ B` entails `B ∧ A` in propositional logic is via a semantic proof. You've probably seen these done via tables (often called "truth tables") like so:

<table style="width: auto">
  <tr>
    <th>A</th>
    <th>B</th>
    <th>A ∧ B</th>
    <th>B ∧ A</th>
  </tr>
  <tr>
    <td>T</td>
    <td>T</td>
    <td style="text-align: center">T</td>
    <td style="text-align: center">T</td>
  </tr>
  <tr>
    <td>T</td>
    <td>F</td>
    <td style="text-align: center">F</td>
    <td style="text-align: center">F</td>
  </tr>
  <tr>
    <td>F</td>
    <td>T</td>
    <td style="text-align: center">F</td>
    <td style="text-align: center">F</td>
  </tr>
  <tr>
    <td>F</td>
    <td>F</td>
    <td style="text-align: center">F</td>
    <td style="text-align: center">F</td>
  </tr>
</table>

Implicit in this table is the meaning of `A` and the meaning of `B`. To make this explicit, they denote something called a proposition. Semantically, propositions are these objects that can be either `T` or `F` (Sometimes read as *"true"* and *"false"* respectively). `T` and `F` don't appear anywhere when we discuss the syntax of propositional logic, they only appear when we ask something like "what does `A` mean?"

We say that `A ∧ B ⊨ B ∧ A` because when we ask what `A ∧ B`, and `B ∧ A` means, any time `A ∧ B` is `T`, so is `B ∧ A` (More generally, we can understand `⊨` here as a relationship between specific rows and columns on a truth table). This is pretty easy in this case because we can exhaustively look at every possibility: All four of them!.

{%aside()%}
Maybe you think the answer to a question like "what does `A` mean?" should be something like "A stands for 'grass is green'". In philosophy, it's common to pick linguistic propositions that we know the truth of instead of picking `T` or `F` directly. Really, what you're doing is picking a line from the table above. Or as another example, lets say `A` stands for *"I will never win the lottery"* and `B` stands for *"pigs can fly"*, you can think of picking a linguistic proposition that we don't know the truth of as either nonsense or as a way to look at two lines in the table above. This case is semantically interesting because `A ∧ B` is `F` regardless of how we interpret `A`. This habit of matching propositional symbols with linguistic phrases is only done for historical reasons or to motivate learning. The underlying semantics is about `T` and `F` regardless of the linguistics you use as a stand-in.
{%end%}

So it turns out that in propositional logic, you can always discover whether `A ⊨ B` by building a truth table. So we don't need a proof system, though a proof system is still nice as once your propositions get more complicated, the rows of these truth tables grow exponentially.

### Proofs using a Proof System

The second way you can prove that that `A ∧ B` entails `B ∧ A` in propositional logic is via its proof system. 

I'm not going to go over proof rules. I'm assuming that even if you've forgotten the specifics, you remember generally how they were used. Instead, I'll ask you to notice that we don't contend with `T` or `F` at all while using a proof system. This is because the end-game of a proof system is to cleverly pick rules that only let you assume something to denote `T` **or** to write something that must denote `T`. If you do this, you can magically know what your truth table looks like for every line where your assumptions are `T` (without ever writing the table out).

<table style="width: auto">
  <tr>
    <th> Line # </th>
    <th> Proof Line </th>
    <th> Proof rule that makes this line legal </th>
  </tr>
    <td> 1. </td>
    <td> A ∧ B </td>
    <td> Assumption </td>
  </tr>
  </tr>
    <td> 2. </td>
    <td> A </td>
    <td> ∧ elimination on line 1 </td>
  </tr>
  </tr>
    <td> 3. </td>
    <td> B </td>
    <td> ∧ elimination on line 1 </td>
  </tr>
  </tr>
    <td> 4. </td>
    <td> B ∧ A </td>
    <td> ∧ introduction on line 3 and line 2 </td>
  </tr>
</table>

We say that `A ∧ B ⊢ B ∧ A` because we have a proof system where if we assume `A ∧ B`, there exists some number of legal lines after which we can legally write `B ∧ A`. This is a purely syntactic game. This becomes an interesting game to us because we've designed it so that it can only generate propositions for which `⊨` also holds. We have a term for this relationship. Having this property where `A ⊢ B` implies `A ⊨ B` makes a proof system {%emph()%}sound{%end%}. (Perhaps less interestingly; for propositional logic it is also the case that `A ⊨ B` implies `A ⊢ B`, which makes propositional logic {%emph()%}complete{%end%})

### Propositional Proofs in General

In propositional logic, the proof system approach is often a much more concise way to prove a proposition. For example, `A ∧ B ∧ C ∧ D ∧ E ⊨ B ∧ C ∧ D ∧ E ∧ A` takes a 32-line truth table, while `A ∧ B ∧ C ∧ D ∧ E ⊢ B ∧ C ∧ D ∧ E ∧ A` takes a 4-line proof (It’s a twin to the 4-line proof from above where `B ∧ C ∧ D ∧ E` replaces `B`).

Consider also that there are logics (like predicate logic) where if you tried to create a truth table, you would be required to write an infinite number of lines. The real power of a proof system makes itself known under such circumstances.

The other cool property of a proof system is that as long as we agree to the same rules, anybody can check a proof and confirm that the rules have been followed. The idea that "there's no special intuition required, anybody can mechanistically check the rules" is what ends up making proofs written in a proof system machine checkable. You can write a program that can check the correctness of any proof.

So even in logics where you cannot "write out the truth table," and for which there's no way to know when to stop searching for a proof... at least you can rest assured that if you ever do find a proof, you and others can certainly verify that the proof is correct.

### Type Systems

So what does all this propositional logic have to do with Type Systems and with Types? 

Well, type systems and proof systems have a lot of similar properties. They both have a clean separation of syntax and semantics, they both offer a purely syntactic set of rules, that anybody (even a program) can check. They both value the relationship between `⊢` and `⊨` so that the rules for what you can do syntactically translate well into the semantics. (Few programming languages actually boast a completely sound type system, though they do still value making the distinction as clear as possible)

Where the proof system above is just there to enforce tautologies, type systems exist to enforce abstractions. Specifically to do so in a machine tractable way. What does that even mean? Well, first lets get back to propositions so we can talk about about Types themselves.

# What are Propositions in Propositional Logic?

We're going to keep coming back to the mathematical substrate I was familiar with before types. It's how I started to build intuitions on the topic, but more importantly for this blog the idea that types and propositions are related is a deeply researched topic. We'll benefit from scratching the surface a bit. I'd like to take a quick detour into what a proposition is because talking about the meaning of things which themselves might denote meaning is more difficult than it first appears.

So when I open my undergrad textbook, the definition of a proposition in propositional logic is as follows:

> A proposition is a statement that can be either {%emph()%}true{%end%} or {%emph()%}false{%end%}; it must be one or the other, and it cannot be both.

To help buttress this definition, I may give some examples of statements which are or are not propositions.

The following are propositions:
  - The reactor is on.
  - The wing-flaps are up.
  - John Major is prime minister.

whereas the following are not:
  - Are you going out somewhere?
  - 2 + 3

This definition leans on a lot of other definitions and intuitions that don't actually surface in propositional logic at all. For starters, you'd be excused for thinking propositions are linguistic expressions, but philosophically the English sentence "Snow is white" denotes the same proposition as the German sentence "Schnee ist weiß" even though the two sentences are not the same. A more nuanced definition may say something about propositions being "the sort of thing that declarative sentences denote".

This is all rather abstract and you can write a PhD Thesis on how linguistic expressions do/have embedded and deviate from formal logic. That sends us down the wrong rabbit whole. Really, propositions in propositional logic are the building blocks of a sort of syntactic game that gets interesting and applicable once you imbue it with meaning.

Syntactically, there are a set of base propositions. For example the elements of `{P, Q, R}` are propositions because we've defined them to be. Then there are a small set of logical connectives and rules for how they're to be used to juxtapose propositions. So the elements of `{¬, ∧, ∨, →, ↔}` are logical connectives. I won't spell out the rules for how to determine a well-formed formula. For example: `P → Q`, `P → ¬Q`, and `P ∨ ¬P` are all well formed but `P Q ∧`, `P ∧ ∧ Q`, and `¬∧P` are all not well formed. Then we perform the neat trick of saying that any well formed formula is a proposition because we define it to be.

So assuming you learned the rules for a well-formed formula and you're armed with a set potentially infinite base propositions, you can simply say that propositional logic propositions are exactly these things. In a sense you can stop here. There are infinitely many of them, but you can create and verify every possible proposition and that's the end of it. You've answered the question: "What are Propositions in Propositional Logic?" as best as it can be answered.

What comes next, the "giving them meaning" part can be done in any number of ways. Most of the semantics we **could** give this system serve no purpose. For example, I can get outright silly with it and say something like "all propositions denote my left pinky finger". I make the syntactic game above somewhat silly because no matter how I puzzle together big complex propositions, all they'll ever do is denote my left pinky finger. That's not helpful.

So instead, let me quote Wikipedia:

> propositions are often modeled as functions which map a possible world to a truth value. For instance, the proposition that the sky is blue can be modeled as a function which would return the truth value {%emph()%}T{%end%} if given the actual world as input, but would return {%emph()%}F{%end%} if given some alternate world where the sky is green. 
> - [Wikipedia <sup>(link)</sup>](https://en.wikipedia.org/wiki/Proposition)

Such functions deterministically tell us how to interpret any proposition as {%emph()%}T{%end%} or {%emph()%}F{%end%} and imbue the entire enterprise with a structured meaning that is interesting. 

As a cool aside, if you view {%emph()%}T{%end%} as the singleton set and {%emph()%}F{%end%} as the empty set, you can see the set theoretic semantics of propositional logic as a generalization of this semantic interpretation of predicate logic. So you can define the functions above using truth tables (the way I did earlier) or set operations like unions, intersections and such. The approaches are the same (up to isomorphism).

# About Types

Types are a syntactic tool for abstraction that don't have meaning other than to add structure to the question of whether some piece of syntax is well formed.

Typically a type theory will have a few base types and then a few type-forming rules. Then types are straightforwardly defined as the (potentially infinite) hierarchy of types that can be generated. This is defining types by just looking at the syntactic game that creates them — seem familiar?.

So really, what's the point of the syntactic type game loosely described above? Types as they're discussed in the more advanced type theory circles have a direct categorical semantics. A friend of mine jokingly says "category theory is general abstract nonsense" (based on how prominent representatives of the mathematical community reacted to its introduction), so we're going to shy away from that for the moment.

One way to model the semantics of types of a simple type theory is as sets of values. To quote:

> "Types” denote nonempty sets of values; they are used to restrict the scope of variables, control the formation of expressions, and classify expressions by their values. 
> - William M. Farmer <sup>[(link)](http://imps.mcmaster.ca/doc/seven-virtues.pdf)</sup>

When you open a Java Class file and see:

```Java
int a = 5;
```

or a Rust file and see:

```Rust
let a: i32 = 5;
```

{%emph(c="blue")%}int{%end%} and {%emph(c="blue")%}i32{%end%} are types and in this case and both denote a {% emph(c="blue") %}32-bit signed two's complement integer{% end %}. The variable `a` is restricted in scope to exactly one of {%emph(c="blue")%}int{%end%}'s 2<sup>32</sup> (or 4294967296) possible inhabitants. There's some loose notion under which `a` denotes an element of {%emph(c="blue")%}int{%end%} as though {%emph(c="blue")%}int{%end%} were a set. If I had written `int a = 5.3,` I would have received a compilation error saying the expression is not well typed. The error would say something to the effect that the solver in the type checker failed to discover a way to unify the expression. In other words, there's no meaningful way to assign a floating point number to a variable meant to hold an integer.

Type systems and type theories use types as a syntactic tool for abstraction, but they don't need to be so embedded to be useful. Consider Hungarian notation, which is an identifier naming convention that prefaces variable names with their data type. For example, the {%emph()%}b{%end%} prefacing {%emph()%}bBusy{%end%} denotes a {%emph(c="blue")%}boolean{%end%} and {%emph()%}nSize{%end%} is an {%emph(c="blue")%}integer{%end%}. This was popularized in a language called BCPL because BCPL has no data types other than the machine word, nothing in the language itself helps a programmer remember variables' types. 

Other untyped languages hold such information in the comments or through other conventions. It's important to note that just like assembly or BCPL still have types in the margins, typed languages also continue that trend. Developers can and do use the type {%emph(c="blue")%}integer{%end%} as a bit-field or the type {%emph(c="blue")%}string{%end%} as tags to approximate a disjoint union.

At their core, types seem like a very simple, but frustratingly abstract thing. They have a somewhat liminal nature, existing in many forms directly in syntax — where rules inductively define well typed syntax — and also in the meta-language — where we leave it as an exercise to the reader to understand why some expressible syntax is not necessarily meaningful.

For example, Rust's type system infers every variable below to be an {%emph(c="blue")%}i32{%end%}. If I were writing a chair-based library for millions of developers to use, I would probably employ Rust's type system to ensure such code never compiles. If I'm writing a quick script, I may just stick with {%emph(c="blue")%}i32{%end%} and be careful about what I write.

```rust
let chair_colour = 15761536; // #F08080 HTML standard Hex colour code
let chair_height = 458; // millimetres

// The following is almost certainly a bug
let syntactically_legal_nonsense = chair_colour + chair_height;
```

# The Naive View of Types in Computer Science

> [A] computer is unable to discriminate between for example a memory address and an instruction code, or between a character, an integer, or a floating-point number, because it makes no intrinsic distinction between any of the possible values that a sequence of bits might *mean*
>
> [...]
>
> Assigning a data type, termed typing, gives *meaning* to a sequence of bits such as a value in memory
> - [Wikipedia <sup>(link)</sup>](https://en.wikipedia.org/wiki/Type_system)

Consider the following sequence of bits:

```
10000001
```

What could this sequence of bits represent as a value?

 - If that's an unsigned 8bit integer, then it could represent a `129`.
 - It also could represent any constant transformation of an 8bit integer:
   - Absolute negative value: `-129`
   - Thousands: `129,000`
   - Even numbers: `258`
 - If that's a signed 8bit integer:
   - It could represent a `-1`
   - If it is in 2s complement then `-127`
 - If the bit that carries the sign isn't the standard one, there's a whole other set of numbers this could represent.
 - It could be an array of two unsigned 4bit integers: `[8,1]`
 - The value may not be an integer at all. Getting more outlandish, it might be a bit-map for items in an adventurer's backpack. In which case this might encode two specific key items (item-id `0` and `7`) out of eight key items an adventurer will need to find during their quest! 
 - This value could be a memory address on a **very** small embedded system, or perhaps the system pads this with zeros to interpret it as a memory address.
 - This could be a machine instruction code of some sort.
 - This could be a path through a binary tree.
 - The above considerations also depend on the "endianness" of the memory layout as well.

You can see the problem. You cannot tell the type of this value by inspecting its bits. Giving those bits a type is the process by which we imbue them with meaning. It doesn't matter that assembly has no means to annotate types and no automated way to check them, any programmer using assembly must do that work manually — they can not reasonably choose to avoid it. For example: it's never desirable to multiply two memory addresses together, so an assembly programmer does their best to avoid doing so. Failing in this task isn't a type error within the language, but it is still a type error conceptually and will result in an incorrect program.

Programmers need to reason about the types of the data they are working with in order to ensure that their programs are correct. 

Here's the catch! Programs are themselves data whose 0s and 1s are imbued with meaning and which developers must be able to reason about.

# Types are about Abstraction

The naive view of types is that:

> Assigning a data type, termed typing, gives *meaning* to a sequence of bits such as a value in memory.

While that is a way to motivate the use of types, this is not typically how types act while programming. It also doesn't really line up with the earlier definition of types denoting nonempty sets of values. The main thing to remember about types, is that they are a tool for abstraction. The easiest way to understand this is to consider how often a developer even sees the sequence of bits that represent the values they use. Especially in the basic nominal cases, developers leave the bit-fiddling to low-level operations.

In general, programmers consider even the most fundamental types to have an existential representation. A well behaved representation exists (because it must), but any other well behaved representation would do just as well. Many Java programmers have never checked (and it has never mattered) whether their {%emph(c="blue")%}int{%end%} is represented in 2's compliment or not. The type system forms an abstraction boundary so that so long as the API of an {%emph(c="blue")%}int{%end%} doesn't change, the representation is largely irrelevant <sup>[1]</sup>.

Instead of "giving meaning" to a sequence of bits, types can be seen in some ways as the opposite. They describe a program that's provably generic over the representation, which means a Java Class doesn't describe a program, but instead a whole set of programs (at least one program for each possible representation of {%emph(c="blue")%}int{%end%}). It just so happens we pick binary representations that work well with our hardware.

In some languages there are pragmatic reasons (say performance/memory) to care about the representation or some part of the representation. Because of its pedigree as a systems language, Rust has 12 different integer types, though they differ on whether they are signed (support negative integers) and how much memory they require, they don't differ on any other representational decisions such as which bit is the most significant, which bit is signed, or whether signed numbers are in two's complement. Even here, the types act to abstract exactly those choices developers want to stop caring about.

{%aside()%}
### You can probably skip this aside:

<sup>[1]</sup> Okay, it's been pointed out to me that this is perhaps not the best example since logical and arithmetic shifts expose the representation of Java's `int`. You can have a well typed Java program where the equality of two expressions differs based on representation. I think this is just a semantic trap. If we tried to formalize what it means when I say "other well behaved representation would do just as well," it would be something like an isomorphism between representations and their given operations. A logical **binary** shift is not isomorphic to a logical **decimal** shift, but you can implement a logical binary shift on a decimal representation just fine. In Java, any of the math operations provided to the language are implemented somehow based on the type's representation, but this is opaque to the language itself.

The `integer` type found in a proof assistant (like *Lean*) would be much more resilient against such concerns than Java's `int`. In a proof assistant `integer` and its operations are defined within the language and not handed over. Setting aside verbosity/performance, this shows it's possible to do better. At the end of the day, however, type systems have hard choices to make around the halting problem or concerns such as hardware crashing during a process. While many type-level abstractions are not, I suspect there will always be some type-level abstractions that are leaky (You can "see through" the abstraction if you know how to mess with the edges just right).

This is a whole can of worms and there's more to say on this topic. Let's leave that for another time.
{%end%}

# A More Nuanced View on Types

To develop a nuanced view of what types you need to understand them in the context of how and why they're used. What motivates the use of types? A simple answer might be "so that the compiler can calculate how much memory some data takes and how to represent the data in memory so that I don't have to do it myself". To support this line of reasoning, languages that have a type system always give you a way to build more complex types from simpler types. Lets run with an example, most languages allow you to define a new product type called a pair from any other two types: `∀ A:type B:type | (A, B):type`. This reads "The pairing of any two types is itself a type". A use for this is that the compiler can understand how a type can be represented based on its construction. For each way you can build up more complex types (like `∀ A:type | [A]:type`, or `∀ A:type | Vec<A>:type`), you can equip the compiler with rules to let it understand what to do with that new type.

There's a deeper structure here, because just like predicates and propositions become useful in the context of a logic that declares how they can be juxtaposed with various operators (syntax) and a model for how to evaluate any legal syntax to it's meaning (semantics). Types become useful in the context of a theory that declares how they can be juxtaposed with various operators (syntax) and a model for how to evaluate any legal syntax to it's meaning (semantics). You're abstracting something more interesting than "memory layout" when you build types and let the compiler figure out the rest.

I don't think the following is necessary to develop a deeper understanding of types, but if you've taken a course or done some reading on {%emph()%}first-order logic{%end%}, then I recommend perusing [this paper (link)](http://imps.mcmaster.ca/doc/seven-virtues.pdf). It's a paper that hopes to recommend teaching a simplified type theory to undergrads. Assuming you have a bit of background with first-order logic, the paper is very accessible. A highlight is where it shows that although simple type theory has none of the propositional connectives or quantifiers, they can all be easily defined in simple type theory using function application, function abstraction, and equality. I'll let the paper speak for itself, but to me the power of Simple Type Theory is that is has a very uniform syntax, but hidden within a hierarchy of types is the full power of higher-order predicate logic.

The theory presented in the paper is type theory boiled down to its essence. It is convenient for study, but it is not highly practical for use. This is where extensions to the theory and macro-expansions and such pick up the slack.

# In Conclusion

The thing to consider then, is that when a computer scientist is using {%emph(c="blue")%}Int{%end%} to label a variable, they can be seen as abstracting away the representation of an {%emph(c="blue")%}Int{%end%}'s value as a series of bits, or as annotating their program with something akin to a logic. What's been done when annotating a variable with an {%emph()%}Int{%end%} is very simple, but as you build complex types from simpler types and construct tuples, lists, sets, etc you're building up more complex logical expressions that let you abstract away the representation of increasingly complex data. You can use function abstraction to view programs themselves as sequences of bits that need abstracting. This lets a developer create boundaries across which the way in which entire parts of your program are represented is abstracted.

In functional programming languages with relatively expressive type systems, it's very common to see the sorts of choices made in imperative languages being encoded as types instead. That way instead of writing out `if, else if, else` statements, you generate a value of some type and by cleverly encoding how these types are allowed to be composed, you're on you way to ensuring certain bad sequences of choices (now types) are not possible to create without a type error. Carefully crafting and letting a type system enforce such invariants for you matters more as software seize, number of developers, and magnitude of maintenance increases.

At a minimum you're adding a logic system that sits on-top of your program and describes some part of it (for example: Java, C#, TypeScript, Go, etc). At the extreme end, you've realized the depths of the {%emph()%}Curry–Howard Correspondence{%end%} and you're a developer writing executable math proofs (for example: Idris, Agda, Coq, Lean, etc).