+++
title = "decouple game from engine"
date = "2023-10-25"
description = "The pros and cons of keeping your game code decoupled from your game engine."
tags = [
    "programming"
]
+++

*[Good design is easier to change than bad design.](https://pragprog.com/tips/)*

This time-honored programming adage is especially pertinent to the development of video games, where "finding the fun" constantly changes the requirements of the code.

One technical decision that is difficult to change is the choice of game engine. When you use Unity, you couple your code to MonoBehaviors. When you use Unreal, you are tied to Actors. To change the engine would effectively call for rewriting the entire game.

However, the benefits of modern game engines cannot be understated. They represent countless developer-hours of platform support and tooling to empower artists and designers. It's usually the right choice to use one for your game.

## why would you want to change your game engine?

You might be doubting the premise that being coupled to a game engine is that problematic. After all, the architecture of game engines usually facilitates rapid prototyping. You can create modular components that are easily thrown out or reused for new purposes. This seems to fit the requirements for "easy to change".

Alas, modern game engines are constantly evolving products. Sometimes, they evolve in ways that don't align with the needs of the game. Perhaps the engine team has announced many new features, neglecting to support existing features that your game depends on. Maybe the company has demonstrated that they are willing to [retroactively change the pricing structure for shipped titles](https://www.gamesindustry.biz/unity-adding-a-fee-for-each-time-a-game-is-installed). Either way, in exceptional circumstances, you might find that being tied to a particular game engine is no longer tenable.

## how do you make switching engines less painful?

My proposal is this:

> Prioritize separating gameplay code from engine code.

In an ideal world, all of the core logic for your game resides in a separate library that is completely decoupled from your game engine. Using a feature from the game engine would be *opting in* to coupling for that specific area of the game.

For example, you could use the game engine simply as a hardware abstraction layer and rendering engine. Switching engines would only require converting the rendering parts to the new engine, leaving the rest of the code unchanged.

Using this workflow, the game engine utilizes the game library as a plugin, with a thin layer on top acting as the glue between the engine and the library API. There are a couple of ways to interface between the engine and the game library:

1. **Poll**: the engine layer will query the game library for the current simulation state, and then reflect that state in the engine. Polling should be a quick operation that does not modify internal game state.
2. **Host**: the engine layer provides callbacks to the game library that control aspects of the game engine. In a way, the game library is a freestanding OS, and the game engine provides the capabilities of the host.

These two methods of interfacing are not mutually exclusive; both should be used as necessary.

Polling is generally preferred over providing a host API because it creates a clear barrier between the game library and the engine. The game library maintains some internal state with no knowledge of who will read it. Compare that to the host interface, which directly involves the engine layer during simulation of the game state. This pushes the game library out of the safety of its sandbox.

The host interface is best used to provide features that a freestanding OS is not capable of, like memory allocation, logging, or input detection.

## why aren't games already doing this?

There are certainly games that are using this approach, it's just not well documented. More importantly, decoupling the game engine and the game logic comes with a cost, just like any decision.

Here are some reasons why you would **not** want to split game engine from game code:

* The risk of a game engine no longer being suitable for production is so low that it is acceptable to be heavily coupled to a game engine.
* Engines are self-contained ecosystems. By splitting gameplay code away from the engine, you're losing out on valuable tooling that the engine provides. (In other words, the game library is somewhat like a "black box" for the engine layer.)
* There is a performance cost for replicating state between the game library and the engine layer. Instead of one combined object, there might be a game state object in the game library, and a render object in the engine layer.
* Iteration time. The round-trip time for making a change to the game library and seeing the result in engine might introduce too much friction.

I have already discussed how the decoupling will make it easier to switch engines, if necessary. Here are some other factors to consider:

* More flexibility in how you approach game programming problems. Being coupled to the Actor or GameObject model can add a lot of unnecessary complexity.
* Running the game in headless mode becomes trivial. It's also easy to run multiple simulations at the same time.
* Opportunity to introduce new tooling or programming languages, as long as they can statically or dynamically link to the game engine.
* Unit testing becomes quicker and easier in a smaller, independent game library.
* Multiple "frontends": if you think of the game library as the "backend", the game engine becomes the "frontend" that can be swapped depending on the platform. For example, you might use a game engine when targeting desktop platforms, but roll your own frontend for mobile platforms and the web.

## what does it look like in practice?

I made a [Tetris clone](https://github.com/bootrako/tetris) that has the following structure:

* A **sim** library that runs a simulation roughly following the rules of [SNES Tetris](https://meatfighter.com/nintendotetrisai/).
* Two **host** projects:
    * A *console* app that runs Tetris on the command line.
    * A *godot* app that hosts the Tetris simulation in a [Godot engine](https://godotengine.org/) project.

Both host projects use the same Tetris simulation library as their backend. The simulation API mostly uses the polling interface. There is a small host interface for allocation, panicking, and reading input.

Feel free to take a look at the code and see how I approached separating game logic from game engine. This was my first attempt at this kind of structure - I would do things a bit differently if I were to start over. Overall though, the code represents the core idea.

## final thoughts

Ultimately, I think that there is a lot of merit to keeping gameplay code in its own sandbox, but I doubt that commercial games will adopt this workflow. If anything, many more companies are abandoning custom engines and adopting Unity or Unreal.

I hope that the concept is at least entertaining to think about from a game architecture standpoint. I will continue to experiment with this concept in my personal projects.

Farewell, and thanks for reading!