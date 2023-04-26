+++
title = "Writing a Hanabi Mod for Tabletop Simulator"
date = 2023-04-21
draft = false

[taxonomies]
categories = ["Experience"]
tags = ["Lua"]

[extra]
toc = true
keywords = "Lua, TTS, Tabletop Simulator, Project, Experience"
+++

# Preface

Roughly a year ago, I wrote a scripted Hanabi Mod for Tabletop Simulation. Here is [the link](https://steamcommunity.com/sharedfiles/filedetails/?id=2793392461). This is a bit of a retrospective on that experience. I talk about Lua, Tabletop's Scripting API, and a bit about my journey programming with them.

# First, Some Thoughts About Lua

Lua is a fast, light-weight, embeddable scripting language. It was designed for configuration and small automation scripts, though you really can't ignore that it's been one of **the** de-facto scripting language for games for decades now. Titans like World of Warcraft, Minecraft, & Roblox all use Lua as their scripting language to develop mods.

## Lua is Small 

{% emph() %}Both physically:{% end %}

> The source contains around 30000 lines of C. Under 64-bit Linux, the Lua interpreter built with all standard Lua libraries takes 281K and the Lua library takes 468K. 
> - Lua: About [(link)](http://www.lua.org/about.html)

{% emph() %}And conceptually:{% end %} I felt like I had a grasp of both the language and it's standard library in an afternoon. Of course, as such things go, sometimes it feels like there's always a little more to learn. Lua does still have all the depth, nuance, and eccentricities of a general purpose programming language.

## Lua is Simple

It offers relatively few abstractions and those that it does offer must be used explicitly.

As an example, consider JavaScript wherein you have named functions, anonymous functions, and arrow functions as well as important details about what a function's context/receiver is (What gets bound to the `this` keyword) depending on which type of function you're dealing with.

In Lua, you have only anonymous functions full stop. (though you can get back a method-style syntax with the colon operator `:`).

As is standard for dynamically typed languages, Lua offers a small set of types and no means to define your own. Lua has — for the most part — leaned into this simplicity and avoided building any complex object/class/type hierarchies (though the option is there via matatables and metamethods). 

Consider Lua's array-hashmap mashup type — {% emph(c="blue") %}tables{% end %}:

> Tables are the main (in fact, the only) data structuring mechanism in Lua, and a powerful one. We use tables to represent ordinary arrays, symbol tables, sets, records, queues, and other data structures, in a simple, uniform, and efficient way. Lua uses tables to represent packages as well.
> - [Programming in Lua (link)](https://www.lua.org/pil/contents.html)

## Lua is Mature

Because Lua been around since 1993, the community and resources for the language are fairly large. To me, its friendly and enthusiastic community is a **major** selling point for the language. 

Lua isn't without its pain-points. Default global scoping, strange table length semantics, no unicode support, limited pattern matching support, and unfortunate return statement semantics are just some of the baggage Lua carries today.

That being said, considering its design goals, speed, simplicity, and the ease with which it can be embedded as a scripting/plugin language, Lua has done just about everything right. I have enjoyed the simplicity and consistency of the language.

# Programming for Tabletop Simulator

One way to think about Tabletop Simulator (TTS) is as a light-weight game engine, allowing you to build scenes filled with 3D assets and interact with them. Most of the game development primitives are present, though they're typically incredibly simplified and geared toward the physics/tabletop experience. 

It's fun. The immediate feedback is welcome and enjoyable when things are working. You don't need to understand any of the finer details of game development to get started. If you want to run a few little scripts here and there, react to certain inputs in simple ways, then the process of doing so with TTS is simple. 

**However**, if you want to build anything reasonably complex, you can do so but TTS either doesn't doesn't offer much help or even gets in your way. Without the active, friendly discord community around modding TTS, I would not have bothered to finish the Hanabi Mod. There are a lot of interactions at play for which the documentation is generally silent: **"If you know, you know, otherwise good luck."**

Here are a few examples that I remember perplexing me at the time: 
- If I want to display a custom UI image floating perpendicularly (at a 90° angle) above a card, I need to multiply the z-axis by a bunch since the custom card object comes pre-compressed on that axis. This works but causes pixelation which you can correct for by stretching your image in Photoshop instead.
- Certain properties of an object are accessible only via their XML/data table so you can't take the recommended route of spawning a custom Object and then provide the custom content for it after spawning it by calling `setCustomObject()` (As an aside, be sure to wait for the physics engine to fully spawn the object before trying to update it). Instead you must prepare an object data table representation which you can derive from another object by calling `getData()` on it, and manipulating the resulting table.
- There are a thousand and one ways for *globally unique identifiers* — GUIDs — to become out of sync with your script's expectations during development. So either be unerringly careful or create a less brittle mechanism by which to identify your game objects.
  - This is especially true if you try to do anything like automate updates/settup/testing for your mod while developing.
  - I used a combination of naming conventions and asset filesystem locations. Then Git can track changes and the scripts are robust against asset updates so long as filenames stay consistent. Not a perfect solution, but a quick one.
- The Table Object is subject to a whole host of unwritten rules, but if you just delete it and use some other custom asset as a stand-in for your table, that all becomes significantly easier to manage.
  - For example, getting the Table to display a custom image with the right proportions was so frustrating that I eventually put it on a board with the right size, removed interactivity, and floated the board just so to make it look like the table.

and so on and so on... the list is long, though it becomes manageable as you keep writing and working with it. The best advice I can give here is to get on forums or discord and ask questions.

# My Experience Programming Hanabi

For me, creating a scripted Hanabi with Tabletop Simulator (TTS) + Lua was fun from a gaming perspective but a relatively joyless development experience. 

TTS doesn't provide the sort of immediate feedback loop that a scripting language like Lua thrives with. There's a tension between the *game-settup, table-building, drag & drop* nature of TTS and its scripting. Often this means that testing some interaction requires you to reload your scripts and then perform some in-game action — spawn, move, click, deal, ect. Tabletop comes with a lot of tools designed so that many typical board game systems are effectively "pre-scripted". This is nice though as your begin creating custom scripts you find yourself needing to work around many of he quirks of these systems.

## Missing Static Types

This one is mostly a matter of preference.

I'm accustomed to programming alongside a type checker. Programming well with a dynamic language requires a certain skill/focus that I'm not well practised with. Without a type system to keep me in check, I'm capable of creating a mess rather quickly.

My development style for this game was very exploratory. When I started, I wasn't really sure what features I wanted and how they would fit together. At first this seems like a good fit for a dynamically typed language like Lua, but the result is that too much time ends up being spent hunting down refactoring bugs leaving less time to experiment with system interactions.

- In a language like Haskell you can re-architect a system. If you've separated out your domain logic (which is where Haskell's purity excels), by the time you've satisfied the compiler there's a gloriously compelling chance everything works as expected the first time you run the refactored program.
  - If you're not sure your idea for how to re-architect your system will even work, this is slow as the "test the happy path first, then worry about making it robust later" style of programming is more difficult. 
  - If you have a clear vision for how your update/new feature will work, a little extra effort removes most of the technical debt associated with a new feature or refactor. 
- In a language like C# (another common game-dev plugin language), you'll have to hunt down a few bugs as the type system so often doesn't know the difference between a total and partial function. There's also bound to be some edge cases around ordering of effects you hadn't considered in your refactoring.
  - It's easy enough to use dynamic variables to more or less ignore the type system when prototyping, though this creates a different kind of technical debt. Post-hoc typing can be difficult as types often encode cross-cutting concerns that may require you to reorganize code to keep your types cleaner/manageable. 
- With Lua, you're better off performing a re-write than trying to refactor for anything as intrusive as re-architecting your system. This can be abated with a good testing suit, but TTS makes writing such a thing extremely laborious if not outright untenable (This isn't Lua's fault, of course). Many of the bugs I caught while play-testing with friends where introduced because of a seemingly harmless refactoring rather than some ill-considered business logic.
  - Progress comes in leaps and bounds. The temptation to simply spaghetti-code your way across the finish line is pretty high.

When I'm in programming mode, I'm somehow able to hold large chunks of a program in my mind. **However**, the moment I context switch away and come back some time later, I find it difficult to re-create the same mind-state. To combat this issue, I work to keep my game logic localized. To the best of my ability, I try to rely on the idea that any block of code which has no hidden dependency is a boon to the continued development or future maintenance of my software.

When it makes sense, attaching scripts to physical in-game objects that can encapsulate and manage their own state is nice. To some degree, this is a simplified version of the standard Object Oriented game-dev design you encounter making games with Unity, Unreal, Godot, etc:
- Hanabi's score-display is a good example of an isolated system like this.
- In general, it seems like game systems don't cleanly attach to in-game scene entities

The TTS event handlers allow for local reasoning as well. With events, you can view game interactions as messages between systems. If you keep messages themselves immutable, you can maintain a clean separation between systems. Unfortunately, event handlers are fairly limited. This is because the TTS scripting API treats almost everything — Players, Objects, and their properties — as global mutable state. Lua's impoverished type system means it's up to the developer to first understand and then forever remember all the interactions at play. The choices made by TTS are orthogonal to the design of Lua, but the fact remains that Lua doesn't offer any tooling to correct course.

## Lua's Coroutines

Here's the thing... In hindsight, I should have stuck with Lua's coroutines. 

When I first toyed around with them, I didn't like the pattern by which you convert TTS's callback-style API into one that smoothly interoperated with Coroutines. By the time I'd seen that this could be done quite nicely, I'd already taken another route and was no longer interested in the size of a refactor that switching back would require.

Here's my solution. A function that flattens nested callbacks:

```Lua
function bindCallback(callback_thunk, bind_fn)
    return function(callback)
        callback_thunk(function(...)
            local thunk = bind_fn(...)
            if thunk ~= nil and type(thunk) == "function" then
                thunk(callback)
            end
        end)
    end
end
```

The general principle here is that you can compose APIs that use callbacks. This means instead of ending up with a deeply nested callback-hell situation (for which old-school JavaScript is so famous), you'll only ever need to be nested at most one level deep. The `bindCallback` function effectively separates everything into "what came before" and "what comes next". You could think of it as a lightweight multi-valued promise in a single function. In that sense, this would act a bit like a promises `.then` method.

This made wrapping the TTS callback API super simple, though on the other hand:
- Using this function isn't always the most ergonomic. 
- Hiding control flow isn't a common Lua idiom, so it might hinder collaboration (for this project: not a big deal)
- This doesn't have a great API for early returns or errors. This doesn't pair that well with TTS's abounding global mutable state
- The simplicity of this approach makes it feel like a half-measure between creating a *Reactive Extensions* style transducer or using Lua's idiomatic and fairly powerful coroutines.

## Global UI

Hanabi's Global UI system is the stand-out feature of the Mod. It has a custom view for each player and allows players to give hints that interact directly with the cards in player's hands.

TTS allows you to build a retained GUI using XML and then to update it dynamically based on events in game. I quickly realized that attempting to model the UI this way would be error prone. For one, each user's state must stay synchronized regardless of the state that the game is saved or loaded from. Also, TTS doesn't save any changes made to the UI’s XML between play sessions. 

I used an MVU architecture (Elm-style *Model View Update*) to simplify the process. This was, I think, one of the big successes of the project as I rarely found myself hunting down bizarre GUI bugs. The amount of game-state to manage for Hanabi is fairly contained, so MVU's monolithic nature isn't a downside.

# Final Thoughts

Building Hanabi was fun, though by the time it was ready, many of my friends weren’t playing much TTS anymore. I felt the project was a good intro to some game development concepts. It was also nice to produce a software artifact that non-programmers could understand and enjoy.