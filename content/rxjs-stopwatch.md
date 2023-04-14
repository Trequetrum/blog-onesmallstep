+++
title = "An RxJS Stopwatch Implementation"
date = 2023-04-13
draft = false

[taxonomies]
categories = ["Code"]
tags = ["TypeScript", "RxJS"]

[extra]
toc = true
keywords = "TypeScript, RxJS, Programming"
+++

# Stopwatch

### Preamble:

Back in the spring of 2020, I ran face first into the stack overflow equivalent to the Baader-Meinhof Phenomenon. I answered a stack overflow question about creating a stopwatch using RxJS and found myself referring back to variations on that exact question every few weeks throughout the summer. I'm putting it here now because it seems intuitively like a task that a streaming library should be able to handle easily. I have never found an answer that clicked for me. Perhaps somebody else will discover or point me at a clever solution that I hadn't considered.

I think RxJS streams push-based nature make them a poor fit for the problem. 

I'm also including it because I think it has some pedagogical merit for a beginner looking to sharpen their skills a bit.
- It uses `defer` to capture state in a closure
  - Each subscription creates it's own local state
  - Operators like `retry` behave as expected
- It uses `scan` to incrementally build data
  - `reduce`, `scan`, and `expand` are friendly operators that are often overlooked
- It uses a higher order operator — `switchMap` — to manage observable lifetimes
  - Mastering `mergeMap`, `switchMap`, `concatMap`, and `ExhaustMap` is nessesary to be proficent with Reactive Extensions
- It builds a custom RxJS Operator
  - Understanding how to create an `OperatorFunction` generally means heading toward an understanding a whole host of interesting concepts that RxJS uses liberally. These include currying, function composition, reducers (aka: fold, accumulate), and finally transducers (composable higher-order reducers).

### The Problem

The crux of the problem is this: 

- Our input is a stream that emits a series of "RUN/PAUSE" or "RESET" actions over an arbitrary span of time.
- If the input stream `error`s or `complete`s, then so should the output. 
  - Once the output is done, all memory should be released to the GC — for example, there should be no ongoing timers.
- The watch has two states:
  - <mark>Running</mark>: The output should emit a number every `interval` milliseconds.
  - <mark>Paused</mark>: There should be no output.
- The watch starts <mark>Paused</mark> and switches state each time the source emits "RUN/PAUSE"
- The Output's emissions are numbers:
  - The first number to be emitted is a zero
  - If the source emits "RESET", the next number emitted is a zero
  - Otherwise, the emitted number is always the previous emitted number + 1.

The most accurate solutions just use the system time. Doing so can avoid the sharp corners of the JS event loop. I think that for the sake of just learning and using RxJS, those concerns add incidental complexity. To that end, there's a relatively succinct solution you can implement by recursively calling `setTimeout`. For a third party auditing such code, understanding the mix of callbacks, `setTimeout()`, `clearTimeout()`, timestamp calculations and so on would not be trivial. We're hoping RxJS can modularize the logic such that a reader familiar with the library could reasonably understand it on a first or second reading. There shouldn't be too much logic that needs to be understood in the context of the broader solution, so that if you understand each part you naturally come to an understanding of the whole.

### A Solution

Here is a solution which leans on RxJS' switchMap operator to manage an internal stopwatch timer. We also use scan to track the watches current state for us. In general, the solution is short enough that it's pretty straight forward.

What I don't like abotu this solution is how `count` is managed. It breaks up an otherwise very modularized solution. The value is initialized, incremented, and reset to zero all on disparate lines of code. The only way to understand what is happening with this value is to understand the entire solution. Which is a shame because everything else can be understood a few lines at a time. It's a fairly contained little piece of state and it seems like the natural way to manage this. On the other hand, `defer` adds a bunch of clutter and exists only to add in a little bit of state. It simply feels like there should be a clean solution that doesn't mutate anything.

The solutions I've thought up that don't mutate state end up being messier to an extent which I don't think makes it worth it. For example, I don't think it's worth doing something like using a long running interval/timer/system-time and remembering pauses via an offset.

```TypeScript
type StopwatchAction = 'RUN/PAUSE' | 'RESET';

function createStopwatch_static(
  control$: Observable<StopwatchAction>,
  interval: number
): Observable<number> {
  return defer(() => {
    let count = 0;

    return control$.pipe(
      scan(
        ({ running }, action) =>
          action == 'RUN/PAUSE'
            ? { running: !running, reset: false }
            : { running, reset: true },
        { running: false, reset: false }
      ),
      concatWith(of({ running: false, reset: false })),
      tap(({ reset }) => { if (reset) count = 0; }),
      switchMap(({ running }) =>
        running ? timer(0, interval).pipe(map((_) => count++)) : EMPTY
      )
    );
  });
}

// Create an OperatorFunction that can be used in an RxJS pipe
function createStopwatch(
  interval = 1000
): OperatorFunction<StopwatchAction, number> {
  return control$ => createStopwatch_static(control$, interval);
}
```

### StopWatch in Use

Control a stopwatch with DOM events to set fields on the DOM.

```TypeScript
// The user clicking buttons in the UI creates the input stream
merge(
  fromEvent(startBtn, 'click').pipe(map((_) => 'RUN/PAUSE' as const)),
  fromEvent(resetBtn, 'click').pipe(map((_) => 'RESET' as const))
).pipe(
  // Operator function turns StopwatchActions into numbers
  // The default interval is a second, so we don't set it
  createStopwatch()
  // If the stopwatch emits an error, re-create the stopwatch
  retry()
).subscribe(seconds => {
  // Edit the DOM to display the current seconds
  secondsField.innerHTML = seconds % 60;
  minuitesField.innerHTML = Math.floor(seconds / 60) % 60;
  hoursField.innerHTML = Math.floor(seconds / 3600);
});
```