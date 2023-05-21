---
categories:
  - image
  - javascript
  - emscripten
aliases: /image-color-whell-generator.html
subtitle: Turn any image into a Hue/Luminosity color wheel
title: Image color wheel generator
date: 2014-02-16
---


[Here is a little side project][charmony] I did, that allows to quickly get an
idea of the color distribution of an image in the Hue / Luminosity [color
wheel][color-wheel].

This can be used by designer to check if an image follows some specific color
scheme, like using only complementary colors.

I wrote the code in C, and compiled it in JavaScript with
[emscripten][emscriten].

I am planning to write a following post to show how I intend to use this tool
for a new video game project I am working on.


[color-wheel]: http://en.wikipedia.org/wiki/Color_wheel
[charmony]: http://www.noctua-software.com/charmony
[emscriten]: https://github.com/kripken/emscripten/wiki

![an image](/assets/imgs/charmony-example.png)
