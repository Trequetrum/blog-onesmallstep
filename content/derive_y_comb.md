+++
title = "Deriving the Fix-Point Y-Combinator"
date = 2023-07-27
draft = false

[taxonomies]
categories = ["Programming"]
tags = ["Intro"]

[extra]
toc = true
keywords = "Syntax, Semantics, Logic, Math, Foundation, Programming, Language, Lambda Calculus, λ"
+++

# Preface

Why would you want to know how to derive the Y-Combinator? Well, mostly for interest's own sake. Modern languages all ship with built-in constructs for recursion. It's not likely you'll ever be required to do something like this.

Curry's presentation of the fixed-point combinator for λ-Calculus (sometime in the late 1930's, I can't find a concrete source) is one of a select few of those early cornerstone results that then went on to shape the world of computing. It's a common misconception that electrical computers begat — or at least motivated — study into computation, complexity analysis, universal decidability, and so on. There's a storied history of logicians delving into these topic long before anyone imagined we would be teaching silicon how to think.

Curry's Y-combinator is both a very elegant formal definition of recursion, and an entrypoint into understanding why developing bug-proof software is systemically difficult.

# Prelude

This blog entry roughly follows an imaginary sequence of thoughts that one could potentially have; the net result of which would be discovering something very much like the Y-combinator — a variadic extension, perhpas? It's not that important what we call it. We'll jump right in with Python. We're going to start with a very simple function and a contrived restriction, then keep taking little steps to abstract a bit at a time.

What we're about to do is feed some of the ideas from λ-calc into a simple python program. I don't beleive writting Python as λ-calc is a either a recipe for success or a very pythonic thing to do. We're treating Python more as a means to actively play around with pseudocode than any attempt to be abidingly Pythonic. Python’s dynamic nature and relatively unobtrusive syntax make it a good language to use for this. In the off chance you want to copy some of the code and play around with it until it makes sense, this should be a fairly straight forward task.

I also think Python's runtime semantics are intuitive to a lot of programmers. So let’s jump in!

# An anonymous version of factorial

Our first step here is going to be to try to puzzle out how to write an anonymous recursive function. 

Without recursion, this task is generally quite simple. Here is a function named `double`, to apply some value to the function we first reference it by name and apply a value.

```Python
double = lambda n: n + n
double(5)
```
```Output
10
```

Of course, if I want to define the same function without naming it, I can do so. However, If I want to call it, I must call it in-line. Like so:

```Python
(lambda n: n + n)(5)
```
```Output
10
```

Why restrict ourselves to anonymous functions? Well, nearly every example of recursion you'll see performs the self-referencing step by name. By untangling this single restriction we can both arrive at a deeper understanding of recursion and derive the Y-combinator. Rather neat, no?

There is a lot of content out there that starts by introducing λ-calc basics and going from there. I chose this sort of sideways appraoch because as far as I know it is a relatively novel way of understanding this cool bit of computing history.

As a formal definition of recurison λ-calculus' Fix-Point Combinator doesn’t much care about how other mathematical objects like numbers or propositions are represented. I think we can get a reasonable distance toward understanding a version of the Fix-Point Combinator via the semantics of Python without worrying about  λ-terms, how variables are introduced, what makes a legal α-conversion or β-reduction, etc. I contend we don’t need any of that for what's described here.

What we do need, however, is the single restriction that stops us from using Python's built in call-site resolution to perform recursion on our behalf. Once we've shown that it can be done, we'll start using names again to keep the code going forward a little cleaner and easy to understand.

Here is the function I want to write anonymously:

```Python
factorial = lambda n: 1 if n==0 else n * factorial(n-1)
factorial(5)
```
```Output
120
```

Assuming `factorial` is just like `double`, we could just try dropping the name and calling it in-line:

```Python
(lambda n: 1 if n==0 else n * factorial(n-1))(5)
```
```Output
NameError: name 'factorial' is not defined
```

Of course, this doesn't work. Since `factorial` referred to itself, removing the name messes with the internals of the implementaiton. So this is the first puzzle: can we define factorial using this simple recursive definition without relying on a global namespace where factorial is already defined?

What we **can** try to do rely on a common programming trick. If you're ever in a position where you've removed some global value (as is sometimes good coding practise), but you want to keep a function that depends on that value, you can pass that value into the function as a parameter instead.

```Python
USER_PREFIX = "User:"
def select_user_repr(user):
    return f"{USER_PREFIX} {user.f_name} {user.l_name} ({user.age})"

# Becomes

def select_user_repr(prefix, user):
    return f"{prefix} {user.f_name} {user.l_name} ({user.age})"
```

Of course, you'll still need to define `prefix` somewhere before calling `select_user_repr`, but how you architect your software to accomplish that doesn't matter to `select_user_repr` anymore (which makes it a bit more readily reusable).

So if `factorial` isn’t in the global scope, maybe we can pass it in somehow instead.

```Python
(lambda f, n: 1 if n==0 else n * f(n-1))(????????, 5)
```

So now we have an anonymous function, but it takes a parameter that we're not sure how to fulfill. This next bit is admittedly a bit of a leap. What goes where the `????????` is sitting above? Since it's a recursive call, the function we want is the same function itself! Can we do that?

If we could use names we might try something like this:

```Python
r = lambda f, n: 1 if n==0 else n * f(n-1)
r(r,5)
```
```Output
TypeError: <lambda>() missing 1 required positional argument: 'n'
```

Oh! Oh yeah, if we pass in `r` as `f` here, then we'll be required to meet the {%emph()%}API{%end%} of `r`. Remember that we updated `r` to take two arguments. We know what they are and they’re now both in scope so we can just include the new first argument.

```Python
r = lambda f, n: 1 if n==0 else n * f(f, n-1)
#_____________________________________^______
r(r,5)
```
```Output
120
```

That seems to work. If you can mess around with the code so far and figure out why this worked, you’ll have managed the most difficult part of figuring out the Fix-Point Combinator.

Of course we've named `r` here which wasn't supposed to be allowed. 

However, we're now in a better place than we were before because `r` doesn't depend on itself explicitly. This means the inlining trick we used with `double` will work with `r`. We just need to copy the definition of `r` twice.

```Python
# 1. r = lambda f, n: 1 if n==0 else n * f(f, n-1)
# 2. r(r,5)
# Substitute r from line 1 in-place of both r in line 2
(lambda f, n: 1 if n==0 else n * f(f, n-1))((lambda f, n: 1 if n==0 else n * f(f, n-1)), 5)
```
```Output
120
```

It's a bit of an eye-sore, but it works! We're going to keep going with `r` for now, but just remember we can expand `r` as shown above.

# Factorial, auto-doubling

When we look at where we've come so far, we have an anonymous function and it works but it's rather annoying to always need to pass in a second identical version of the function each time we want to call it. Instead, we'd like to have a function where we can just pass it a number and get the factorial of that number back as a result.

This is pretty easy, as it turns out. We'll parameterize the `5` in the snippet above into a new function and then call that with `5`.

```Python
r = lambda f, n: 1 if n==0 else n * f(f, n-1)
(lambda n: r(r,n))(5)
```
```Output
120
```

So now we can call this in-line with a simple number and the function calls itself with the correct parameters and we're off to the races again.

We've come to the point where we've accomplished the first task that we set out to accomplish. We've created an anonymous version of `factorial`. We can call it a few times with different values and compare it against the standard non-anonymous version just to convince ourselves this really works.

```Python
# Recursive starting function
factorial = lambda n: 1 if n==0 else n * factorial(n-1)
# Anonymous rewrite
r = lambda f, n: 1 if n==0 else n * f(f, n-1)

def fact_test(num): 
    return factorial(num) == (lambda n: r(r,n))(num)

list(fact_test(n) for n in [5,7,10,13,20])
```
```Output
[True, True, True, True, True]
```

We're just mucking about, so seeing this output is good enough for now. 

We could stop here, this really isn't too bad. Though of note is that right now we've hand-written a wrapper for `r`. If we write a new recursive function, then we'll need to write another wrapper as well.

For example:

```Python
#greatest common denominator
gcd = lambda a, b: a if b % a == 0 else gcd(b % a, a)
# Anonymous rewrite
r = lambda f, a, b: a if b % a == 0 else f(f, b % a, a)

(lambda a, b: r(r,a,b))(42, 28)
```
```Output
14
```

This wrapper function isn't really doing anything special, so instead of a wrapper for `factorial` and another wrapper for `gcd`. We can generate wrappers automatically for any similar recursive function expression. 

First, lets describe what it means to be a "*similar recursive function expression*". There are two criteria.

1. You must take the recursive call of the function as the first parameter.
2. You must pass forward the first parameter as the first parameter again on any recursive calls.

Loosely, your function must look like this:

```Python
# Notice that f is the first arg and that f calls itself with f as the first arg
def any_name(f, *any_other_args):
    # Any code you want, probably a base case
    # ...
    # Recursive call
    f(f, any_computed, values, here)
    # Any code you want
    # ...
    return something
```

We're going to write a function `mock` (as a reference to the *"To Mock a Mockingbird"* puzzle book) that generates these wrappers automatically for us. This is a fairly common pattern in computer science. You might hear terms like Adapter, Decorator, or Proxy used in the Object Oriented software dev world. At its simplest, you take some functionality as input and return some functionality that wraps the input with some extra behaviour.

Consider `no_op_wrapper` below. It returns a new function that acts the same as the given function but is just one level deepeer in the generated call-stack. There's a level of indirection but no change in behaviour.

```Python
def no_op_wrapper(f): 
    def wrapper(*args, **kwargs):
        return f(*args, **kwargs)

    return wrapper

no_op_wrapper(lambda x: x + x)(5)
```
```Output
10
```

`mock` is just like `no_op_wrapper`, only it adds some observable behaviour. Now the given function is applied to itself first before the call is made.

```Python
def mock(f): 
    def wrapper_with_f_applied(*args, **kwargs):
        return f(f, *args, **kwargs)

    return wrapper_with_f_applied
```

You can write this a bit more concisely using Python's lambda expressions:

```Python
mock = lambda f: lambda *a, **kw: f(f, *a, **kw)
```

This lets us do the recursive wrapping trick for any function written in this form.

```Python
mock( # Factorial
    lambda f, n: 1 if n==0 else n * f(f, n-1)
)(5)
```
```Output
120
```
```Python
mock( # GCD
    lambda f, a, b: a if b % a == 0 else f(f, b % a, a)
)(42, 28)
```
```Output
14
```

We got the right answers, so `mock` seems to be working for us.

# Pre-applied Recursion

In every one of our anonymous recursive form of functions, the second requirement is that we're always going to need to call `f` with itself:

```Python
# Factorial
lambda f, n: 1 if n==0 else n * f(f, n-1)
#_______________________________^^^______

# GCD
lambda f, a, b: a if b % a == 0 else f(f, b % a, a)
#____________________________________^^^______
```

This is a small, but repetitive step. Its possible to get this step wrong and create some pretty bizarre control flow. 

It's also a rule that should seem a bit suspicious. The first rule is what lets us create a recursive call, but the second rule is just an extra bit of bookkeeping. We should think about whether this is accidental or essential complexity. Consider that we've already created a function called `mock` that removes our need to call `r` with itself. There doesn't seem to be any reason (in principle) why we could't use `mock` to call `f` with itself too.

Really, as a point of pride, we want this step gone. The puzzle here is: how?

Consider the following where we want to pass in `mock(f)` instead of `f`

```Python
mock(
    lambda f, n: 1 if n==0 else n * mock(f)(n-1)
)(5)
```
```Output
120
```

Well, we can use the same trick where we wrap the input and return a modified version. Let's take this idea and expand it out for factorial to see it at work. Here we add in a wrapper around `r` that doesn't do anything yet:

```Python
mock = lambda f: lambda *a, **kw: f(f, *a, **kw)
r = lambda f, n: 1 if n==0 else n * f(f, n-1)

mock(             r      )(5)\
    ==                       \
mock(lambda f, n: r(f, n))(5)\
    ==                       \
120
```
```Output
True
```

Of course a wrapper that does nothing isn't useful, but we can modify this slightly by altering the wrapper's `f` {%emph()%}before{%end%} passing it to our recursive inner function. Which will mean that factorial will no longer need to perform that step for itself.

```Python
mock = lambda f: lambda *a, **kw: f(f, *a, **kw)
# Update r so it isn't passing f forward anymore
r = lambda f, n: 1 if n==0 else n * f(n-1)

mock(lambda f, n: r(mock(f), n))(5)
```
```Output
120
```

In doing this, we've crested the hill. 

The expression we're wrapping no longer requires `f` to pass itself forward on each subsequent call. The rest of the work should more or less be standard stuff to clean up the work we've done so that instead of working for just the given recursive expression, we can create a function that will work for **any** similar recursive expression.

Lets update the rules for what you must do to be a similar recursive expression

1. You must take the recursive call of the function as the first parameter.
2. ~~You must pass forward the first parameter as the first parameter again on any recursive calls.~~

That feels nice.

Let's parameterize `r` by creating a function that takes in a function like `r` and does all the recursive work we've just figured out. Remember that while the `r` above just takes a parameter n, we don't make that assumption when we parametrize. Instead we'll replace `n` with any {%emph()%}args{%end%} or {%emph()%}kwargs{%end%}

```Python
mock = lambda f: lambda *a, **kw: f(f, *a, **kw)

def fix(r):
    return mock(lambda f, *a, **kw: r(mock(f), *a, **kw))
```

Lets try it out!

```Python
fix( # Factorial
    lambda f, n: 1 if n==0 else n * f(n-1)
)(5)
```
```Output
120
```
```Python
fix( # GCD
    lambda f, a, b: a if b % a == 0 else f(b % a, a)
)(42, 28)
```
```Output
14
```

Which gets us to a really nice place. Using fix, we can create any recursive function anonymously if we just include `f` as the first parameter and call it exactly the same way we'd make our recursive calls usually.

```Python
fact =     lambda    n: 1 if n==0 else n * fact(n-1)
# becomes
fact = fix(lambda f, n: 1 if n==0 else n *    f(n-1))
#--------------------------------------------------------------
gcd  =     lambda    a, b: a if b % a == 0 else gcd(b % a, a)
# becomes
gcd  = fix(lambda f, a, b: a if b % a == 0 else   f(b % a, a))
```

# fix is the Y-Combinator

Fix is actually the Y-Combinator with some `*a` and `**kw` thrown in to allow us to define our recursive functions in a more python-familiar variadic functions style.

So, if the steps we took along the way didn’t throw you for a loop, you can now derive the  Y-Combinator for yourself! Exciting!

# Appendix

Let's take our definitions, drop the `*a` and `**kw` stuff and see what it looks like. In λ-calc, doing this is akin to something called an {%emph()%}η-reduction{%end%}. In Python, this reduction isn't as straight forward. So while the code below is Python-like, don't take it too seriously. We play a bit fast a loose with the syntax here so that if you see this stuff in the wild, you may recognise it.

This is really the least important part of the article :)

```Python
mock = lambda f: lambda *a, **kw: f(f, *a, **kw)
# η-reduction (Drop extra arguements that are just passed through)
mock = lambda f: f(f)
# α-conversion (Rename variables)
M = lambda x: x(x)

fix = lambda r: M(lambda f, *a, **kw: r(M(f), *a, **kw))
# η-reduction (Drop extra arguements)
fix = lambda r: M(lambda f: r(M(f)))
# expand the first and second M
fix = lambda r: (lambda x: x(x))(lambda f: r(((lambda x: x(x)))(f)))
# β reduction (Apply f to the second M)
fix = lambda r: (lambda x: x(x))(lambda f: r(f(f)))
# β reduction (Apply expression on the right to the first M)
fix = lambda r: (lambda f: r(f(f)))(lambda f: r(f(f)))
# α-conversion (Rename variables)
# This is Y as given by Curry
Y = lambda f: (lambda x: f(x(x)))(lambda x: f(x(x)))

# λ-Calc notation
Y = λf.(λx.f(xx))(λx.f(xx))
# Or the version we need in eager evaluation languages like python to 
# force lazy execution. So this is a bit closer to the fix function we
# derived in this article (Think of z like "*a, **kw")
Z = λf.(λx.f(λz.xxz))(λx.f(λz.xxz))
  = λf.(  λxz.xxz   )(λx.f(λz.xxz))
```