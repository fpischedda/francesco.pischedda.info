.. title: Evaluating Rust for game development
.. slug: evaluating-rust-for-game-development
.. date: 2018-05-06 18:19:55 UTC+02:00
.. tags: rust gamedev programming
.. category: 
.. link: 
.. description: 
.. type: text


Evaluating Rust for game development
====================================

There is no doubt that Rust is gaining popularity as a system development
language and most of the time games and game engines are built using languages
such as c or c++ which were born with the same target in mind, being able to
access all the capabilities that a machine has to offer with the minimum
runtime overhead.

In the last years, powerful engines and tools have offered a pletora of bindings
for different languages like C#, Lua, Python, Javascript and so on with great
results; for example Unity have had and continue to have a great success
providing a powerful platform to build game on so why taking the risk and effort
to use such a low level language to build a game?

As an "old school" developer I prefer to be in control of my software development
stack and often, instead of creating the game that I have in mind, I start
creating an engine to build my game on...but why?
That's because it is fun! Or because for some simple game it seems to be easier
to create a small library (or extend some old code) instead of using a powerfull
solution like Unreal Engine, or and or and or...

I have started creating games again and again using high level languages like
Python, Javascript, Lua and even the good old c++; in general I really like to
create prototypes with python but it is not easy to distribute your games and
almost impossible (or reaay hard) to target mobile platforms.
Another option is to target the browser with the growing set of available engines
but again, targeting mobile platforms can be a problem and targeting consoles is
almost impossible.

Rust on the other end is a very young player in the field but with its
ability to target many architectures I think that starting to think about
using it can be a safe bet; there is at least one success story for using rust
for game development and the final result is `encouraging<https://www.rust-lang.org/pdfs/Rust-Chucklefish-Whitepaper.pdf>`.

List of available resources
---------------------------

Right now the best collection of rust game development related project can be
found in the "Are we game yet" `page<http://arewegameyet.com/>`.
this web site maintain a list of frameworks and engines fro the lowest level
flavour to the most complete game engine.

Of all the available solutions `ggez<https://github.com/ggez/ggez/>` (Good Game Easy)
is the one which attracts me the most because I like to create small prototypes
and not having the knowledge overhead required by other full featured engines or
frameworks like `piston<https://github.com/PistonDevelopers/piston>` or
`amethyst<https://github.com/amethyst/amethyst>` it is helpful when experimenting;
it also feels like a microframework to which you can add the components you need
without having to comply with other people choices and taste.
