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

Curry's presentation of the fixed-point combinator for λ-Calculus (sometime in the late 1930's, I can't find a concrete source) is one of a select few of those early cornerstone results that then went on to shape the world of computing. It's a common misconception that electrical computers begat — or at least motivated — study into computation, complexity analysis, universal decidability, and so on. There's a storied history of logicians delving into these topics long before anyone imagined we would be teaching silicon how to think.

Curry's Y-Combinator is both a very elegant formal definition of recursion, and an entry point into understanding why developing bug-proof software is systemically difficult.

# Prelude

This blog entry roughly follows an imaginary sequence of thoughts that one could potentially have; the net result of which would be discovering something very much like the Y-Combinator — a variadic extension, perhaps? It's not that important what we call it. We'll jump right in with Python. We're going to start with a very simple function and a contrived restriction, then keep taking little steps to abstract a bit at a time.

What we're about to do is feed some of the ideas from λ-calc into a simple python program. I don't believe writing Python as λ-calc is a either a recipe for success or a very pythonic thing to do. We're treating Python more as a means to actively play around with pseudo-code than any attempt to be abidingly Pythonic. Python’s dynamic nature and relatively unobtrusive syntax make it a good language to use for this. In the off chance you want to copy some of the code and play around with it until it makes sense, this should be a fairly straight forward task.

I also think Python's runtime semantics are intuitive to a lot of programmers. So let’s jump in!

# An anonymous version of factorial

Our first step here is going to be to try to puzzle out how to write an anonymous recursive function. 

Without recursion, this task is generally quite simple. Here is a function named `double`, to apply some value to the function we first reference it by name and apply a value (Instead of a standard Python function `def`, I’ve assigned a lambda expression to a variable. It just makes the transformation into an anonymous function very simple).

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

Why restrict ourselves to anonymous functions? Well, nearly every example of recursion you'll see performs the self-referencing step by name. By untangling this single restriction we can both arrive at a deeper understanding of recursion and derive the Y-Combinator. Rather neat, no?

There is a lot of content out there that starts by introducing λ-calc basics and going from there. I chose this sort of sideways approach because as far as I know it is a relatively novel way of understanding this cool bit of computing history.

As a formal definition of recursion λ-calc's Fix-Point Combinator doesn’t much care about how other mathematical objects like numbers or propositions are represented. I think we can get a reasonable distance toward understanding a version of the Fix-Point Combinator via the semantics of Python without worrying about  λ-terms, how variables are introduced, what makes a legal α-conversion or β-reduction, etc. I contend we don’t need any of that for what's described here.

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

Of course, this doesn't work. Since `factorial` referred to itself, removing the name messes with the internals of the implementation. So this is the first puzzle: how can we define factorial using this simple recursive definition without relying on some external state where factorial is already defined?

If you’re like me, thinking this through should break your brain a little bit. Understanding the relationship between recursion and looping is a bit more intuitive. If you write a stack to push and pop some state, you can re-write anything recursive as a loop. If you don’t have any loops and you don’t have any built-in recursion, how can you do recursion by just passing functions around?

What we **can** do is try relying on a common programming trick. If you're ever in a position where you've removed some global value (as is sometimes good coding practise), but you want to keep a function that depends on that value, you can pass that value into the function as a parameter instead.

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

So now we have an anonymous function, but it takes a parameter that we're not sure how to fulfill. This next bit is admittedly a bit of a leap. Let’s think through what goes where the `????????` is sitting above. Remember that this is a recursive call, we want to pass in the same function that we’re calling! Can we do that?

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

However, we're now in a better place than we were before because `r` doesn't depend on itself explicitly. This means the in-lining trick we used with `double` will work with `r`. We just need to copy the definition of `r` twice.

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

The parametrization above doesn’t require self-reference, just a function with the same behaviour. For example:

```Python
r1 = lambda f, n: 1 if n==0 else n * f(f, n-1)
r2 = lambda f, n: n * f(f, n-1) if n!=0 else 1
r1(r2,5)
```
```Output
120
```

If you trace what’s happening here, `r1` is actually only called the once. `r2` is then called n-1 times until an answer is found. In a way, `r1`’s primary purpose here is to bootstrap `r2`. The part of `r1` that does the bootstrapping is this: `f(f, n-1)`. You can see how it’s calling `f` with itself, which we know ends up being `r2` with itself. What if `r1` doesn’t do any of the rest and **just** acts to bootstrap `r2`? Something like `r1 = lambda f, n: f(f,n)` should work? Well yes, but there’s another way to understand this.

# Factorial, auto-doubling

When we look at where we've come so far, we have an anonymous function and it works but it's rather annoying to always need to pass in a second identical version of the function each time we want to call it. Instead, we'd like to have a function where we can just pass it a number and get the factorial of that number back as a result.

This is pretty easy, as it turns out. We'll parameterize the `5` in the snippet above into a new function and then call that with `5`.

```Python
r = lambda f, n: 1 if n==0 else n * f(f, n-1)
# r(r,5) <-- pull out the 5 as a param ‘n’ for a new function
(lambda n: r(r,n))(5)
```
```Output
120
```

So now we have an anonymous function can call this in-line with only a number and it will evaluate to the factorial of that number. The function calls `r` with the correct parameters and we're off to the races again.

We've accomplished the first task that we set out to accomplish. We've created an anonymous version of `factorial`. We can call it a few times with different values and compare it against the standard non-anonymous version just to convince ourselves this really works.

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

This wrapper function isn't really doing anything special, all it does is create a new function that's lacking the first parameter of the wrapped function and then calls the wrapped function properly. The only things that change from wrapper to wrapper is how many parameters there are and which function is being wrapped. Instead of a wrapper for `factorial` and another wrapper for `gcd`. We can generate wrappers automatically for any similar recursive function expression. 

First, lets describe what it means to be a "*similar recursive function expression*". There are two criteria:

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

Consider `no_op_wrapper` below. It returns a new function that acts the same (extensionally equal) as the given function. There's a level of indirection but no change in behaviour.

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

This lets us do the recursive wrapping trick for any recursive function expression as discussed above. Lets try it with `factorial` and `gcd`:

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

In every one of our recursive function expressions, the second requirement is that we're always going to need to call `f` with itself:

```Python
# Factorial
lambda f, n: 1 if n==0 else n * f(f, n-1)
#_______________________________^^^______

# GCD
lambda f, a, b: a if b % a == 0 else f(f, b % a, a)
#____________________________________^^^______
```

This is a small, but repetitive step. Its possible to get this step wrong and create some pretty bizarre control flow. 

It's also a rule that should seem a bit suspicious. The first rule parametrizes `f` and is what lets us create a recursive call. The second rule is just an extra bit of bookkeeping. It just re-aligns the API to this new form of a function that takes itself as a first parameter. We should think about whether this is accidental or essential complexity. Consider that we've already created a function called `mock` that removes our need to call `r` with itself. There doesn't seem to be any reason (in principle) why we couldn't use `mock` to call `f` with itself too.

Really, as a point of pride, we want this step gone. The puzzle here is: how?

Consider the following: `mock(f)(n-1)` is extensionally equal to `f(f, n-1)`.

```Python
mock(
    lambda f, n: 1 if n==0 else n * mock(f)(n-1)
)(5)
```
```Output
120
```

Given that this works, the recursive expression for `factorial` doesn't call `f` directly anymore. If we could give the expression `mock(f)` in place of `f`, then we can drop the requirement that `f` needs to call itself as the first argument.

We can use the same trick where we wrap the input and return a modified version. First, lets create a wrapper around our recursive expression `r` that doesn't do anything yet:

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

Next, we add a modification to our wrapper:
 
Of course a wrapper that does nothing isn't useful, but we can modify this slightly by altering the wrapper's `f` {%emph()%}before{%end%} passing it to our recursive inner function. Which will mean that factorial will no longer need to perform that step for itself. Instead of calling `mock(f)` as part of the implementation of `r`, we call `mock(f)` before calling `r`.

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

The expression we're wrapping no longer requires `f` to pass itself forward on each subsequent call. We're now in a similar position as before we wrote `mock`. We need to create this custom wrapper around `r` where we call `mock(f)`. The solution is the same as well. We'll create a new function called `fix` that wraps our new type of recursive function expression.

Here are the updated rules for our recursive function expressions:

1. You must take the recursive call of the function as the first parameter.
2. ~~You must pass forward the first parameter as the first parameter again on any recursive calls.~~

That feels nice.

Here's what `fix` looks like.

```Python
mock = lambda f: lambda *a, **kw: f(f, *a, **kw)

fix = lambda r: mock(lambda f, *a, **kw: r(mock(f), *a, **kw))
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

Let's take our definitions, drop the `*a` and `**kw` stuff and see what it looks like. In λ-calc, doing this is very loosly akin to something called an {%emph()%}η-reduction{%end%}. In Python, this reduction isn't as straight forward. So while the code below is Python-like, don't take it too seriously. We play a bit fast a loose with the syntax here so that if you see this stuff in the wild, you may recognize it.

This really is the least important part of the article :)

```Python
mock = lambda f: lambda *a, **kw: f(f, *a, **kw)
# η-reduction (Drop extra arguments that are just passed through)
mock = lambda f: f(f)
# α-conversion (Rename variables)
M = lambda x: x(x)

fix = lambda r: M(lambda f, *a, **kw: r(M(f), *a, **kw))
# η-reduction (Drop extra arguments)
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