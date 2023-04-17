---
layout: post
title: C++ is not cross platform anymore
categories: C C++ rant
redirect_from: /cpp-is-not-cross-platform-anymore.html
---

As of 2019, C and C++ are the only programming languages that are supported by
virtually any platforms on the market.  For example both iOS and android
support compiling C and C++ directly as part of their official IDEs.

This is one of the reason why I decided to write my voxel editor [Goxel] in
C99: I wanted to be able to run it on Linux, Mac, Windows, iOS, and now I am
working on a Android port as well.

While the code of goxel is mainly C99, there are a few external libraries
that I use that are written in C++.  I compile them directly alongside
the code, so I don't have to care about binary release or ABI compatibility.

In theory this is great, except that recently I get more and more difficulties
making C++ work in a cross platform way.  The C++ that I learned a long time
ago (basically C with some extra syntax for class and templates) has been
updated to a more modern language with the C++11, C++14 and C++17 versions.
Those new versions added features like lambda, variadic, templates, binding
declarations, etc...  The standard C++ library (almost considered a part of the
language) has also seen a lot of changes.

The difficulty comes from the fact that not all C++ features are
implemented by all toolchains.  For each version of gcc and clang (the two most
popular C++ compilers) there is a set of features that are supported, and
it's almost impossible to keep track which one.  Also different versions will
have some subtle differences in how they interpret the specification.

This doesn't matter if you work on a fixed environment: you can just use
whatever feature is available with the toolchain you use.  But if you want
to be able to port your code to other platform, you can only use the smallest
subset of C++ that is supported by all of those platforms.

Because of this I run into troubles in goxel.  Here is an example of build
that worked on my machine, but failed on travis continuous integration server:
[https://travis-ci.org/guillaumechereau/goxel/jobs/553716829](
https://travis-ci.org/guillaumechereau/goxel/jobs/553716829)

One of the errors here is:

    In file included from src/yocto.cpp:56:
    src/../ext_src/yocto/yocto_shape.cpp:1632:42: error: no matching function for
          call to 'size'
      solver.graph.reserve(size(positions) + size(edges));

This seemed to indicate that `std::size` wasn't recognised by clang 7, but
in fact some local tests show me that it is.  Maybe the <iterator> header
wasn't properly included?  Or the compiler didn't try the `size` templated
function from std because some other size function have been defined?
It is really hard to tell, and I couldn't reproduce this on my computer even
with clang 7.

An other error from the same travis failed build:

    src/../ext_src/yocto/yocto_image.cpp:339:9: error: cannot decompose this type;
      'std::tuple_size<const yocto::vec4f>::value' is not a valid integral
      constant expression
    auto& [a, b, c, d] = abcd;

Once again, this is hard to tell how to fix that.  Different versions of
clang and gcc could have different idea of what is an integral constant
expression.

Since all those errors are sent to me by email from travis servers, I have
no way to test them by myself, I can only attempt to fix the code and submit
a new version until it works.

At least with C (or C99), I am usually certain that if the code compile with
one version of the compiler it will with all the others.  There are a few
language extension to avoid (like gcc nested function, of clang blocks)
but that's pretty much it.

The solution is probably to avoid most 'modern' C++ feature and only use
those that have been proved to work well everywhere.  I urge people who write
open sources libraries to avoid using C++ features as much as possible.
Consider using C99 instead.

[Goxel]: https://guillaumechereau.github.io/goxel/
