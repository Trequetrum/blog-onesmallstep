+++
title = "An RxJS Stopwatch Implementation"
date = 2023-04-06
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

Back in the spring of 2020, I ran face first into the the Baader-Meinhof Phenomenon wherein I answered a stack overflow question about creating a stopwatch using RxJS and found myself referring back to variations on that exact question every few weeks throughout the summer. At the time, I created a quick non-public github gist that I could reference whenever such a question appeared. 

I'm putting it here now because it seems intuitively like a task that a streaming library should be able to handle easily but I never found an answer that clicked for me. Perhaps somebody else will discover or point me at a clever solution that I hadn't considered.

I'm assuming some passing familiarity with RxJS, or really any similar streaming library.

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

### A Solution

Here is a solution which leans on RxJS' switchMap operator to manage an internal stopwatch timer. We also use scan to track the watches current state for us. In general, the solution is short enough that it's pretty straight forward.

What I don't like is how `count` is managed. It's a fairly contained little piece of state and it seems like the natural way to manage this. On the other hand, `defer` adds a bunch of clutter and exists only to add in a little bit of state. It simply feels like there should be a clean solution that doesn't mutate anything. 

The solutions I've thought up that don't mutate state end up being messier to an extent which I don't think makes it worth it. For example, I don't think it's worth doing something like using a long running interval/timer/system-time and remembering pauses via an offset.

```TypeScript
type StopwatchAction = 'RUN/PAUSE' | 'RESET';

function createStopwatch(
  control$: Observable<StopwatchAction>,
  interval = 1000
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
```

### StopWatch in Use

Control a stopwatch with DOM events to set fields on the DOM.

```TypeScript
createStopwatch(merge(
  fromEvent(startBtn, 'click').pipe(mapTo('RUN/PAUSE')),
  fromEvent(resetBtn, 'click').pipe(mapTo("RESET"))
)).subscribe(seconds => {
  secondsField.innerHTML  = seconds % 60;
  minuitesField.innerHTML = Math.floor(seconds / 60) % 60;
  hoursField.innerHTML    = Math.floor(seconds / 3600);
});
```