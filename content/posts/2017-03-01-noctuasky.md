---
categories:
  - astronomy
  - stellarium
  - noctuasky
  - emscripten
alias: /noctuasky.html
title: Introducing Noctua Sky
date: 2017-03-01
---


![Noctuasky](/imgs/noctuasky.png)

I just released [Noctua Sky], a new project I am working on.

This is an online planetarium (or sky map) software similar to the open source
project [Stellarium] made by my brother.

The goal of the project is ultimately to provide a set of tools and API for
amateur astronomers to share their observations online.  The first step toward
this is to create a JavaScript planetarium that can run directly on the
browser.

Technically the application was written in C, and then compiled to JavaScript
using [Emscripten].  I already wrote about Emscripten in [an other post](
http://charlie137-2.blogspot.tw/2013/01/voxel-invaders-ported-to-javascript.html)
regarding a video game I made before.

The code is not open yet, but if you are interested in using the JavaScript
client in your own website please contact me.

[noctua sky]: https://noctuasky.com
[Stellarium]: http://www.stellarium.org
[Emscripten]: http://kripken.github.io/emscripten-site/
