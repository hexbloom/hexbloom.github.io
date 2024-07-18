+++
title = "godot first impressions"
date = "2023-10-29"
description = "A retrospective on learning the Godot game engine by building a SNES Tetris clone."
tags = [
    "programming", "godot"
]
+++

I've decided to try out the [Godot game engine](https://godotengine.org/).

While [Unreal](www.unrealengine.com) remains dominant in the AAA scene, not all developers are looking for engines with such high-end features. [Unity](https://unity.com/) has long been the frontrunner for 2D games and games that ship for mobile or the web. Following [Unity's recent missteps](https://finance.yahoo.com/news/unity-responds-huge-backlash-over-131515630.html), Godot has a golden opportunity to challenge Unity in this space.

The best way to learn a new tool is by making something, so I made a clone of the [SNES version of Tetris](https://www.youtube.com/watch?v=-FAzHyXZPm0). It's a small slice of the full game that drops you into the A-Type mode.

You can try it out by clicking the button below! Just a few disclaimers:

* The download size for the game is 28.5 MB
* Only desktop browsers are supported. I tested on Chrome and Firefox.
* These are the controls:
    | Action     | Right Handed | Left Handed |
    | ---------- | ------------ | ----------- |
    | Move L     | Left Arrow   | A           |
    | Move R     | Right Arrow  | D           |
    | Fast Drop  | Down Arrow   | S           |
    | Rotate CW  | X            | Period (.)  |
    | Rotate CCW | Z            | Comma (,)   |
    | Restart    | R            | R           |
    | Mute       | M            | M           |
    | Pause      | P            | P           |

{{< include_html "content/blog/1_godot_first_impressions/tetris_embed.html" >}}

## is godot any good?

Determining if an engine is "good" depends on various factors, the most important of which is your game. Some engines are better for 2D or 3D. I suspect if I tried to use Unreal to recreate SNES Tetris I wouldn't have great things to say about it.

Godot's niche is 2D games, so my expectations are set high. Before I started this project, I was hoping that the engine had these qualities:

* Quick and easy setup.
* Documentation is up-to-date and useful.
* There should be helpful tools that solve common problems for 2D games.
* Engine code is well-structured and can be used to fix bugs.
* Scripts are tightly integrated and have a quick iteration cycle.
* I can deploy the game on my website.

In addition to learning a new engine, I also wanted to try [separating gameplay code from game engine code](/decouple-game-from-engine/). With that in mind, I also wanted these features:

* I can easily integrate third-party code as a plugin.
* Iteration time for plugin code isn't terrible.

While this seems like many requirements, both Unity and Unreal hit most of these bullet points, in my experience. So, how did Godot do?

**Spoiler:** Godot performed excellently and satisfied every requirement, but not without some reservations. Continue reading for a detailed breakdown!

## build system

There are two ways to setup the Godot editor:

1. Download the compiled editor binary.
2. Download the source code and compile locally.

I wanted to be able to debug source code and have the most flexibility in creating a third-party plugin, so I decided to go with option 2.

Godot uses a build system called [Scons](https://scons.org/). It's a Python-based build system that I had never heard of before this project. Admittedly, I wasn't too thrilled about having to learn yet another build system. But installing it and performing a basic build was easy enough.

I was in a unique situation: before I had even downloaded the Godot source code, my gameplay code was pretty much done. I had written a separate [Tetris simulation library](https://github.com/bootrako/tetris/tree/main/sim) and tested it using a [command line renderer](https://github.com/bootrako/tetris/tree/main/host/console).

One of the first things I decided to do was create a plugin that linked to my Tetris simulation. Godot has a couple of approaches here:

1. Create an [engine module](https://docs.godotengine.org/en/stable/contributing/development/core_and_modules/custom_modules_in_cpp.html).
2. Use [GDExtension](https://docs.godotengine.org/en/stable/tutorials/scripting/gdextension/what_is_gdextension.html).

In brief, here are the pros and cons of each:

* Engine modules are static libraries that are linked to the engine source code. If you change the code in a module, you have to recompile the engine. The module itself needs to be C or C++ code, but theoretically the module could be a wrapper around another library written in another language, as long as the library doesn't directly interface with Godot types.
* GDExtensions are shared libraries that are loaded by the engine executable. You can recompile a GDExtension without recompiling the engine. However, shared libraries introduce additional challenges, like being poorly supported on web or mobile platforms. The GDExtension system provides the capability to write bindings to the Godot API for other programming languages.

For Tetris, I decided to go with the engine module approach. I had already written my simulation, so I didn't need fast iteration time. Additionally, I knew I wanted to deploy the game to the web, which doesn't have the best support for shared libraries.

Adding an engine module was pretty straightforward and the documentation was a great starting point. I was pleased with the API to add bindings for GDScript. The engine code is easy to read and helped me diagnose problems when they arose.

That's not to say adding the module was without its problems. I had a particularly tough time wrangling with Scons over the fine details. By default, Scons outputs intermediate files (like .o or .lib files) in the same directory as the source files. In contrast, most other build systems I use have an output directory where all build artifacts are stored.

This becomes a problem when these intermediate files start to show up in the third party library directories. I don't want to add a .gitignore to my engine-agnostic third party library specifically for Godot! The solution to this problem is [variant directories](https://scons.org/doc/1.2.0/HTML/scons-user/x3346.html), but they are unwieldly to use, especially since the Godot engine code does not use them.

Ultimately, I figured out the proper [incantation](https://github.com/bootrako/tetris/blob/main/host/godot/tetris_godot/SCsub), but it took a lot of experimentation and there was not much help online to be found.

On the flip side, there are some nice parts about Scons. I like how all Scons config scripts are just Python files. This makes it easy to throw in print messages to debug the state of the builder during execution. [CMake](https://cmake.org/), in comparison, is a pain to debug when something goes wrong.

Overall, here's how I feel about compiling and adding modules to Godot:
* Documentation was good enough to get going with engine modules.
* The module API is nice and exposing functionality to GDScript is easy.
* Scons has some rough edges, but the tool is generally good.
* Iteration time wasn't that bad. Making a change in my module code took about 30 seconds to recompile.

## editor tools

The Godot editor felt instantly familiar to me, based on my experience with other game engines. I enjoy the explicit split between 2D and 3D. This enables Godot to provide 2D-specific features without detracting from the 3D capabilities of the game.

In Godot, 2D games work in pixel space, which is a welcome change from other engines like Unity. Godot has the best out-of-the-box support for pixel-perfect alignment of sprites and GUI elements of any game engine I've used. I especially appreciated being able to configure separate viewport and window sizes, with automatic [letterboxing](https://en.wikipedia.org/wiki/Letterboxing_(filming)). In Godot this is simply a project setting, while in Unity you would need to download a plugin or use a custom shader to achieve a similar effect (as far as I remember).

In general, the tools for solving my problems were not hidden behind subtools or custom editor windows. Instead, they were simple properties on the objects I was creating.

For example, I added a sprite atlas and wanted to create a prefab that only showed a single string in the atlas at a time. All I had to do was import the texture, add the texture to a Sprite2D object, and set the number of horizontal and vertical frames. When I wanted to change the sprite, I just needed to set the frame index to a different value.

There was no need to create a custom animation asset or go into another window to specify how to cut the atlas into sprites. Most of the time using the Godot editor I felt like I was in the golden path of user experience.

Some other miscellaneous thoughts:

* Configuring input actions that can bind multiple keys to a single input event was built-in and simple to use.
* The editor feels lightweight. No long wait times when initializing the project.
* Initially I was concerned about how heavily Godot leaned into inheritance. While the _types_ of nodes are deeply inherited, in the scene graph you build mostly using composition. Instead of adding components to objects you have parent nodes which can have multiple children.

## scripting

The primary scripting language for Godot is called [GDScript](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_basics.html). It uses a Python-inspired syntax and it's dynamically typed.

The alternative to GDScript is C#, [but you have to compile with the .NET runtime](https://godotengine.org/article/whats-new-in-csharp-for-godot-4-0/). I didn't want to add that extra overhead to my project.

I was initially doubtful that I would want to use GDScript. My early plan was to use GDScript for prototyping and switch to C++ once I had an idea of what I was doing.

By the end of the project, the only Godot C++ I had written was some code to provide GDScript bindings for the Tetris simulation API. Through using GDScript, I gained appreciation for the benefits of having a deeply integrated scripting language:

1. Iteration time is basically zero. Even though compiling C++ code has a modest turnaround time of 30 seconds, with GDScript I can make a change and instantly be testing the code. That 30 seconds adds up over hundreds of changes.
2. Great debugging support that feels similar to debugging native code. It's also helpful to be able to see the state of the scene graph while at a breakpoint.
3. GDScript has fantastic discoverability. Autocomplete is a big help, and you can right click a property and press "Lookup Symbol" to see the class or method definition with documentation. I was able to discover that the String class has a [pad_zeros function](https://docs.godotengine.org/en/stable/classes/class_string.html#class-string-method-pad-zeros) that did exactly what I needed it to, which saved me time having to write the function myself.
4. There is some [static typing support](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/static_typing.html)! Not only can it help [improve runtime performance](https://godotengine.org/article/gdscript-progress-report-typed-instructions/), but it helps with autocomplete and catching errors before you run the code. The built-in editor will display the line color in green if the code is type safe, which becomes its own fun minigame as you work.
5. At the very least, GDScript is a text-based programming language. This is better to me than visual scripting languages like [Unreal Blueprints](https://docs.unrealengine.com/5.2/en-US/blueprints-visual-scripting-in-unreal-engine/): they can be easily diffed and changed without having the editor open.

Even though GDScript performance may be a bit worse than compiled languages, the productivity benefits are a worthy tradeoff.

I have some minor nitpicks. Not everything can be made typesafe (like 2D arrays). Export variables seem a little finicky and would sometimes set themselves to null. But overall, GDScript won me over, and I plan to keep using it for future Godot development. 

## exporting

Exporting, in this context, is packaging the game into its final standalone format on the desired platform. On Windows, it's converting into the final .exe file format. On the web, it's converting into the .wasm, .js, and .html files that the browser serves to the user.

Godot takes a novel approach to exporting. Users are expected to provide a separately compiled version of the engine (called an [export template](https://docs.godotengine.org/en/stable/tutorials/export/exporting_projects.html#export-templates)) to the editor. The packaging process converts the Godot project's resources into the proper format for the target platform and then bundle those resources with the compiled engine version.

If you're building the engine from source, you'll need to also manually compile the export templates. Otherwise, you can download pre-compiled export templates from the Godot website.

Building an export template is generally simple. To start, simply supply `target=template_release` to Scons and it will compile without tools (i.e. without the editor). Since this is the profile of the engine that we want to release with, we'll want to enable optimizations as well.

Things start to get a bit dicey when we want to optimize for size. When exporting for the web, final packaged size should be minimized to save on bandwidth and download times. Godot has some documentation for [optimizing a build for size](https://docs.godotengine.org/en/stable/contributing/development/compiling/optimizing_for_size.html), but even after following all of the steps, the engine build (without game assets) is ~26 MB. There is an open [issue](https://github.com/godotengine/godot/issues/68647) about this, and I think Godot has room for improvement here.

A positive of the export process for the web is the friendly Javascript interface for the engine. It gives you plenty of configuration options that are [well documented](https://docs.godotengine.org/en/stable/tutorials/platform/web/customizing_html5_shell.html). Ultimately I ended up serving the default export html file through an iframe, but I had the option to take a different approach if I wanted.

## final thoughts

I enjoyed making my first game in Godot. I had way more positive comments than critical ones in my post. Perhaps this was biased by the fact that I made a rather simple 2D game. I would imagine the story would be much different if I instead had chosen to make a 3D game, where Godot is not as polished.

One thing to keep in mind is that Godot is an open source project with a tiny amount of resources compared to commercial game engines. There are going to be rough edges and things that don't work. The silver lining is that everything is completely open - you are empowered to figure out solutions and contribute back to the engine.

I will continue to make more games with Godot, and I am hopeful that the engine will continue to improve as time goes on!