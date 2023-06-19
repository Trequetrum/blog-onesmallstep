+++
title = "Dynamic Vs Static Type Systems"
date = 2023-06-10
draft = true

[taxonomies]
categories = ["Types"]
tags = ["Intro"]

[extra]
toc = true
keywords = "Types, Syntax, Semantics, Logic, Math, Foundation, Type Theory, Programming, Language"
+++

This is the second in a series of loosely related posts exploring Types, Type Theory, Type Systems, and some related content from the point of view of a computer scientist.

# Context

The Dynamic Vs Static Type System debate has raged strong since the 90s. It's much like {%emph()%}tabs vs spaces{%end%} or {%emph()%}toilet paper: over vs under{%end%}. There's a lot of hot air and righteous fury you can bring to the table if you so choose. I'm not here to end the debate, I just think it's fun to pick a side and bat for the bleachers.

**So here it is:** *Static Type Systems offer everything Dynamic Type Systems do {%emph()%}plus{%end%} types* <sup>[*]</sup>

<sup>[*]</sup> <sub>at the minor cost of being a bit more verbose.</sub>

# Introducing the Sum Type

You can put type information into the run-time of a program.

The best known (and universally applauded) type that does this is the signed integer. Statically, a signed integer is a sum type. One bit is reserved to tell the system how to interpret the other bits. For example: if the tag bit is a `1`, the other bits represent the absolute 'value plus `1`' of a negative number (The 'plus `1`' means there's no way to represent `-0` as distinct from `+0`). If the tag bit is a zero, the other bits represent a positive number. Like any type, how this is done is down to convention. As long as disparate parts of the system/program agree on which bit is the front-matter for a signed integer, there is no further intrinsic meaning to the choice.

This is a type that requires an invariant not explicit to the construction of the type being upheld. That is, that the front-matter bit faithfully describes what follows. If your system ever mismanages the front-matter, then `7 - 10` becomes `3` instead of `-3` (or `2147483645` instead of `-3` given 2s compliment). There are strict rules for how the system is allowed to interact with such a type in order to maintain this invariant.

This is a pattern that can be used in a broader context and even embedded directly into a language. A programmer can statically make a choice to allocate some extra front-matter to a value which can then be used to discern how to interpret the rest of the value. Any language with a tagged union, variant, variant record, choice type, discriminated union, disjoint union, sum type, coproduct, etc is giving developers the ability to create such types.

Dynamic languages like JavaScript, Python, Lua, etc somtimes take this idea to an extreme and create a massive discriminated union of all their types. Statically, their values can be introspected into any known run-time type. There is a trade-off made here. Every value must be bigger due to this front-matter and the run-time must consistently and repeatedly check the front-matter whenever a typed operation is performed (Modern interpreters/compilers for dynamic languages can use static analysis to infer some static typing and check types a bit less incessantly). Such languages tend to rely on hidden/implicit control flow — for example, throwing an error — to handle all the extra possible run-time errors. From a static perspective, this can feel a bit like a return to untyped languages like assembly, only now with handrails to make testing for errors more friendly.

On the other hand, there are benefits to this approach as well. When programming with a dynamic language your reasoning can be a bit fast and loose since type errors are still caught at run-time. Printing a value can implicitly inspect the discriminated union in which the language encodes all its run-time values. Developers no longer have the issue where there is no way to tell a value's type at run-time (They can blindly assume some standard front-matter). This enables a style of programming where developers can stop reasoning about the possible states of their program and reason only about a state they've seen. While error prone, it's certainly a fast way to accomplish smaller tasks and explains why most scripting languages are dynamically typed.

Here's the thing, any statically typed language with algabraic data types (both sum and product types) can perform this trick as well. You can always write your program to effectively use only a single type, just like Python and JavaScript do it behind the scenes.

# An Example

We're going to write a fully dynamic-style program using a strongly typed static language. Here's how it works, I'm going to model something approximating Lua's dynamic type. I picked Lua because it's simpler and we're here for a proof of concept. I'll then write a short program in dynamic and then typed pseudo-code.

The typed code will be working at a disadvantage. Idiomaticlly typed code can do cool things like "pick the right data-type for a situation," but doing so does not create a fully dynamic-style program. So the typed code must perform all the redudant type checks and such that the dynamic code's runtime system performs for it.

Below, I've picked the very first Python function I found by picking a random page from [https://python.readthedocs.io](https://docs.python.org/3/tutorial/controlflow.html):

```Python
def ask_ok(prompt, retries=4, reminder='Please try again!'):
    while True:
        ok = input(prompt)
        if ok in ('y', 'ye', 'yes'):
            return True
        if ok in ('n', 'no', 'nop', 'nope'):
            return False
        retries = retries - 1
        if retries < 0:
            raise ValueError('invalid user response')
        print(reminder)
```

Idiomatically, with types you might write this as follows

```rust
```