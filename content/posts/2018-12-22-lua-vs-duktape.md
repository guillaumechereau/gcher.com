---
categories:
  - C
  - lua
  - javascript
  - duktape
  - embedded
alias: /lua-vs-duktape.html
title: Lua vs Duktape for embedded scripting language
date: 2018-12-22
---


I recently started to see how to integrate a generic script language into
my voxel editor [Goxel].

I want to use something light and easily embeddable into a C application.  One
obvious choice for this is to use [Lua], but I also wanted to try [Duktape],
a lightweight JavaScript interpreter.  In this post I'll give some of my
conclusions after trying both.

## 1. SIMPLICITY TO EMBED

Both Lua and Duktape are relatively easy to embed into a C application.  They
use the same stack based approach that is both flexible and simple in term of
API.  The C functions you want to expose need to take a stack object from
which you can retrieve the arguments passed to the function.  Output from
functions are also passed into a stack.

Duktape API is almost identical to Lua, so there is not much to compare here.
I tend to find Duktape API slightly simpler, mostly because the library
is provided as a single C file with a corresponding header file.


## 1. DOCUMENTATION

In my opinion Duktape has a better documentation than Lua.  Most of the time
just reading the comments in the header file is enough to find what a
function does.  The online doc is using nice diagrams to explain how each
function will affect the stack.  This is really helpful since it's not always
obvious if a call will pop it's arguments from the stack or not.

Lua's website contains a link to a free version of the book 'programming in
Lua', but it only covers Lua 5.2 and so some of the examples are not working
anymore.


## 2. LANGUAGES

Lua interprets it's own language, while Duktape is just one of the many
interpreters of JavaScript (specifically ECMAScript E5).  Both languages have
some good and bad things in my opinion.  JavaScript is probably the most
popular language these days, so it might seem like the obvious choice.

Things I like in Lua: the meta-object model is quite clever.  The language
uses a special syntax for method call (using colon instead of a dot),
which makes the conceptual model of objects very natural and flexible.  In
comparison JavaScript object support seems poor.  The introduction of Proxy
objects helps a bit at the cost of an other layer of complexity.

One thing I don't like much in Lua is the fact that the arrays are one indexed.
This is specially annoying for Goxel since the scripts will have to index into
multi dimensional arrays for which zero indexed is the only sane way.  As an
example, this is how you would index a 2d image memory block in Lua:

    img[(y - 1) * w + x]

While in JavaScript you simply have:

    img[y * w + x]


As for javascript, in my opinion the most annoying thing is the confusing
coercion rules.  For example in Javascript:

    [1, 2] + [3, 4]

Evaluate to the string:

    '1, 23, 4'

An other problem with Javascript is that as of ES5, there is no easy
way to create multi line strings.


## 3. SPEED

This is where Lua is the clear winner.  In my test scripts, I got results twice
as fast with Lua than what I get with duktape.  For most scripts that is
probably not an issue, but in Goxel I would like to generate large 3D volumes
from parametric functions, and for this the speed of the script interpreter
is going to be critical.


## CONCLUSION

At this time, I am still unsure what I should use, though I tend to think I
will go for Lua.  Duktape seems the most future proof choice, since JavaScript
is so popular, but the performances penalty is quite high.  I thought about
trying other JavaScript interpreters, but it seems most of them are too
heavy for my use.


[Goxel]: https://guillaumechereau.github.io/goxel/
[Lua]: https://www.lua.org/
[Duktape]: https://www.duktape.org/
