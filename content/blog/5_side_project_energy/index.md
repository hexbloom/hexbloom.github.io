+++
title = "side project energy"
date = "2024-02-10"
description = "Discussing how game engines interfere with game programming"
tags = [
    "programming"
]
+++

This topic came up in a thread and I wanted to write about it because I have **opinions**:

> Overall, I don't really love spending my side project energy on figuring out how to cram a game's data model into to a game engine's mix of scene graph/component/node stuff.

## what kind of side project are you making?

A _side project_ is a project that you spend time on outside of normal working hours. Typically side projects are hobby projects, but some may be commercial. Since they are hobby projects, the goal is usually to have fun with it, rather than being the most productive. Some people treat side projects as second jobs, though.

My goal with side projects is _work escapism_. You see, I am a game engineer, which means I spend 40 hours a week fighting against game engines. Game engines are general purpose tools, and in practice what that means is that the tools in the engine get you 80% of where you need to be to ship. The last 20% represent the desperate struggle to make the engine do exactly (or almost exactly) what you need it to do.

For this reason, using a game engine for my side projects has been the last thing I've wanted to do. It just feels like _work_. I figured I could just write little games from scratch and build it up from there.

## making game engines is hard

I've spent an embarrassing amount of time trying to make "games from scratch" work for me. This has usually involved me booting up the Vulkan tutorial for the hundredth time to recreate the same thousand lines of triangle before I eventually hit the cross platform shader compiler complexity wall and give up. For some reason all of my "from scratch" game projects must be cross platform and use minimal dependencies.

One day, a miracle happened and I decided to actually try to make a game instead of a game engine. I don't know what in particular inspired this change - perhaps it was me turning 30 and realizing that I haven't even gotten close to my goal of releasing a game. My brain hack to making a game engine not feel like work was to use the game engine that most AAA game companies would never use, [Godot](https://godotengine.org/).

As it turns out, game engines _are_ useful tools for making games! I was able to make a [Tetris](https://github.com/bootrako/tetris) clone and had a bunch of fun with it. In brash definance, I refused to use the engine's scripting languages for gameplay, and instead wrote the entire gameplay layer in C. That lead me to write my first blog post: [decouple game from engine](https://bootrako.github.io/decouple-game-from-engine/).

## using a game engine without using a game engine

I really do like the Godot game engine. But there's another anti-game-engine factor at play: the desire for the code to feel like my own. I spend all day working on other people's code. I just want to have my own little sandbox where things can be exactly the way I want them to be.

So now that I have gotten over the hurdle of using a game engine for a side project, I am faced with the internal struggle of how _much_ to use the engine. I was happy using the engine as a rendering / input / audio layer, but there's actually a bit of nuance in where you draw the line:

* Do you use the engine's animation state machine tool?
* Do you use the engine's built-in tilemap system?
* Do you use the engine to drive configuration values?

In making Tetris, as well as working on a follow-up _Kirby's Dream Land_ clone, I've learned a lot about where I draw the line for these kind of decisions.

## use the game engine as little as possible

Ultimately, making the decision to keep some gameplay code in a separate sandbox introduces friction into the development workflow.

The first time this became a non-trivial issue is when I was working on animation for _Kirby's Dream Land_. In my engine-agnostic layer, I had some state keeping track of Kirby's internal simulation state: whether he was jumping, falling, or grounded, move speed, etc. I had the Godot engine layer read from this state and update an [animation state machine](https://docs.godotengine.org/en/stable/tutorials/animation/animation_tree.html#statemachine).

Over time, the player state increased in complexity to the point that I refactored it to also be a state machine. Now I had two state machines - one in the gameplay layer and one in the engine layer, and I had to duplicate my work every time I added a new state! My solution was to completely remove the engine animation state machine and instead read the current state from the gameplay layer and play the animation that matched the state.

Another area where I ran into some friction was with the level editor. I decided to use the engine's tilemap editor to configure the level's collision map and place enemies. I wrote an editor script to export the level to a text file that the gameplay layer was able to read and use to spawn enemies. Every time an enemy is spawned in the gameplay layer, the engine layer handles the enemy spawned event and creates the visual representation of the enemy. This part of the process was automatic and I was mostly happy with it. I was able to visualize the level layout much better than editing a text file.

The downside to this approach is that the runtime level scene in Godot was the same as the scene I used to bake the collision data. This leaves a bunch of unused collision tilemap data in the scene file that gets loaded at runtime. Trying to split up the scenes would be problematic, as I want to visualize the collision tilemap along with the rest of the art for the level. Trying to create a level editor that would let me bake collision data, persist the collision tilemap somewhere, and then have it be ignored / deleted at runtime feels just like the kind of work I'm trying to avoid in my side projects.

My overall opinion is that if you want to use tools in the game engine, just use the game engine as intended. Don't do what I'm doing. But if you are going to do what I'm doing (creating a separate gameplay code layer so you can have a happy space to work in), then use the game engine as little as possible to minimize friction between the layers.

## but isn't that just reinventing the wheel?

If you think that sounds like [NIH syndrome](https://en.wikipedia.org/wiki/Not_invented_here), then you would be absolutely correct! Just remember that in your happy little side project space, you can be accepted even if you suffer from such an affliction. Turn back now if that's a dealbreaker for you.

Okay, with that disclaimer out of the way, I would like to point out that since modern game engines are general purpose tools, the tools are often entirely too complex for your use case. As an example, the [tilemap tool](https://docs.godotengine.org/en/stable/tutorials/2d/using_tilemaps.html) in Godot has support for all kinds of specialized workflows: collision, navigation, occlusion, handling multiple texture atlases, etc. For my game, I just wanted a 2D grid editor that had multiple layers.

I'd argue that while there is an up-front cost to writing your own tools for your game, you can drastically lower the scope of those tools by making them minimally feature-complete for your specific game.

What you want the game engine code to handle is the parts of the game that are either too hard to tackle or not interesting enough to spend your time on. For me, I can't afford to get rid of  the engine's windowing, input, or rendering capabilities.

For my next project, I'm going to try to handle almost everything at the gameplay code layer, and at the end of the gameplay update loop, emit a list of commands for the engine to churn through. These commands might look something like this:

* Create sprite
* Move sprite to position
* Set sprite region in texture atlas
* Play sound

The engine can handle these commands and create the necessary scenes / game objects necessary.

And at this point, using a game engine feels a little heavy handed. Perhaps something like [SDL](https://www.libsdl.org/) would be a better fit, especially with a [cross-platform modern graphics API](https://discourse.libsdl.org/t/future-3d-api/33485) coming in the future.

## full circle

Really, my thoughts on this topic overall boil down to this: figure out the minimal set of external code needed to make your game, and write the rest yourself.

* Too much external code forces you to spend a lot of time plumbing it to fit your needs - this feels like work.

* Too little external code can leave you spinning your wheels trying to solve a problem you aren't interested in or is too complex to get done in a reasonable amount of time. 

To remind us of the quote that we started with:

> Overall, I don’t really love spending my side project energy on figuring out how to cram a game’s data model into to a game engine’s mix of scene graph/component/node stuff.

My advice would be to spend your side project energy trying to figure out how to minimize using the game engine instead of trying to adapt the game engine to your gameplay code.