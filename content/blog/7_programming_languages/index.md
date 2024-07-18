+++
title = "on programming languages"
date = "2024-04-14"
description = "What to consider when choosing a programming language for a new project"
tags = [
    "programming"
]
+++

There's a short list of topics that are guaranteed to generate _hot takes_ in
your typical programming roundtable:

* Tabs vs spaces
* Braces on new line or same line
* (new!) Memory safety

For this post, I'd like to discuss another one of these topics:

* **programming languages**.

Perhaps it's a surprise that we haven't converged on a single programming
language to rule them all. Even when you narrow down to a subdomain, like web
development, systems programming, or games, you'll still find a diverse set of
options to choose from.

### programming languages are more than just a feature set

Let's consider two systems-level programming languages by thinking about their
most prominent features:

* Rust has the borrow checker, which prevents common memory safety issues at
  compile time.
* Zig has `comptime`, enabling more expressive type reflection and eliminating
  runtime overhead.

Both of these features sound great. In fact, I would say the borrow checker
seems like more of a game changer than comptime.

But when I use Rust, something doesn't click. I spend most of my time fighting
with the compiler and redesigning my program. Even though Rust is a mature
language, very feature-complete, and has been adopted by major tech companies,
it's just not for me.

On the contrary, I really enjoy using Zig. The language just fades away and I am
able to focus on the problem at hand. Zig is still in early development, there
are constant breaking changes, and it hasn't achieved most of the goals it has
planned. Even so, I would easily choose Zig as my language of choice for my
future projects.

After using both of these languages, I had a realization. While the features of
a language might draw you in initially, long term usage is dependent on
something else entirely.

### programming languages reflect the mindset of the developer

In my opinion, the mindset of the Rust developer is one of correctness. The
borrow checker is the ultimate reflection of this. _You will not be able to
compile your program unless it is memory safe._ Sounds reasonable, if not
preferable to the status quo.

There is a cost to enforcing correctness, though. The compiler becomes a source
of friction. Rust requires a lot of annotations, type traits, and other markup
in order to appease the borrow checker. 

Consequently, prototyping and mocking up code in Rust is the most frustrating
part of using the language. I think if you know exactly what the scope of the
problem is and everything that goes into the implementation, Rust becomes a
great choice. Since I'm a game developer, the core functionality of the program
is often in flux due to the iterative design process. That makes Rust overall a
poor language choice for me. 

In contrast to Rust, Zig has a focus on simplicity. Drawing from C, the simplest
systems programming language, one could argue that several of the "features" of
Zig are omissions:

* no hidden control flow (e.g. operator overloading)
* no macros or preprocessor
* no hidden memory allocations

This makes both reading and writing Zig code very straightforward, and the
learning curve to pick up the language is very shallow if you have any C
experience.

The big additive feature of Zig is `comptime`, which is the ability to call any
function during the execution of the compiler. So, if you wanted to write a
function to build a lookup table based on some complex algorithm, you could do
all of that at compile time. This concept allows for amazing flexibility in how
you design your programs, but does not get in the way of normal development.

Compared to Rust, there's not much to consider when designing a program in Zig.
There's no type traits, lifetime annotations, or borrow checker to get in your
way. You put data into structs just like you would in C. Zig gives you tools to
deal with the pain points, like non-nullable pointers or the `defer` keyword to
cleanup at the end of a scope. It feels like programming in "easy mode".

### use the tools that fit your mindset

It goes without saying that my opinions about Rust and Zig are... _opinionated_.
I won't elaborate further on those languages since there's plenty of material
online discussing the pros and cons of each. The real takeaway I am hoping to
convey is that there is no absolute "best" language out there. With enough
effort, you use pretty much any language for any purpose. In the future, when
considering which programming language to choose for a project, think about what
developer mindset you have and use that to guide your decision.
