---
categories:
  - game
  - noctua
  - procedural
  - graphics
  - opengl
aliases: /noc_turtle.html
commentsUrl: http://blog.noctua-software.com/noc_turtle.html
subtitle: Introducing noc_turtle library
title: Procedural graphics generation in C
date: 2015-06-12
---


I introduce [noc_turtle], a small MIT licence C library to create procedural
graphics in plain C.  I release it in the hope that some people will use it in
their own project, and also to promote my video game [Blowfish Rescue] for
which I wrote it.

The library allows to write procedural rules directly in C, so it is very
simple to embed it in a project and to mix it with custom code.

The name comes from the turtle concept of the [LOGO] language, since it uses
a similar abstraction.

# How it works

Let start with a simple example.  Here is a valid C function, using some
macros defined in noc_turtle.h:

    static void demo_sun(noctt_turtle_t *turtle)
    {
        START
        TR(S, 0.2, SN);
        TR(HUE, 40, SAT, 1, LIGHT, 0.7);
        CIRCLE();
        LOOP(16, R, 360 / 16.) {
            RSQUARE(0, X, 1, S, 0.8, 0.1, LIGHT, 0.2);
            CIRCLE(X, 1.7, S, 0.4);
        }
        END
    }

When we feed the function into noc_turtle, we get the following image:

![noc_turtle demo sun](/assets/imgs/noc_turtle/sun.png)

If you know the [Context Free project], the code should feel familiar.  The
idea is that the function takes a turtle as argument and tell it what to do.
CIRCLE, and RSQUARE are primitives to render respectively a circle and a
rounded rectangle at the curent position of the turtle.  The arguments to
the primitives are a list of adjustments to do to the turtle before rendering
the primitive.  For example the line

    RSQUARE(0, X, 1, S, 0.8, 0.1, LIGHT, 0.2);

tells the turtle to do a translation along x of 1 unit, then scale itself of
0.8 in the x axis and 0.1 in the y axis, then increase its color lightness of
20%, and finally render a rounded square.  The first argument (0) defines the
roundness of the square.  After the operation the turtle returns to its initial
state, that is: the adjustments only affect the current primitive.

The adjustments used in LOOP are applied to the turtle at each iteration of
the loop, so in that case we call the block of the loop 16 times, with each
time a rotation of 22.5 degree.

The TR function (for TRansform) doesn't render anything but allows to change
the current turtle attributes, globally to the function.

# Animations

The first interesting thing is that we can easily create animations by adding
pause in the rendering code.  If we add a YIELD call in the loop of the
previous function, like that:

    static void demo_sun(noctt_turtle_t *turtle)
    {
        START
        TR(S, 0.2, SN);
        TR(HUE, 40, SAT, 1, LIGHT, 0.7);
        CIRCLE();
        LOOP(16, R, 360 / 16.) {
            YIELD(5);   // Pause for 5 iterations.
            RSQUARE(0, X, 1, S, 0.8, 0.1, LIGHT, 0.2);
            CIRCLE(X, 1.7, S, 0.4);
        }
        END
    }

Now we get this animation:

![noc_turtle demo sun animated](/assets/imgs/noc_turtle/sun.gif)

Note that the YIELD here is not pausing the application: under the hood the
library is using coroutines to keep track of where in the programme a turtle
is, so that the function can return and be resumed later from where it was
paused.  If this idea seems weird, you can check [this explanation][coroutines]
by Simon Tatham.


# Multiple turtles

The previous example was rather simple, with a single turtle doing linear
rendering.  We can however use the library for more complex animations.  Just
like a unix process, a turtle has the capacity to fork itself so that multiple
rendering functions can be executed in parallel.  Here is an other example, a
bit more complicated:


    static void spiral_node(noctt_turtle_t *turtle)
    {
        START
        SQUARE();
        YIELD();
        if (BRAND(0.01)) {
            TR(FLIP, 0);
            SPAWN(spiral_node, R, -90);
        }
        SPAWN(spiral_node,
              X, 0.4, R, 3, X, 0.4,
              S, 0.99,
              LIGHT, -0.002);
        END
    }

    static void demo_spiral(noctt_turtle_t *turtle)
    {
        START
        TR(HSL, 1, 100, 0.5, 0.5, S, 0.02, SN);
        SPAWN(spiral_node);
        SPAWN(spiral_node, FLIP, 90);
        END
    }

Here is the resulting animation (I removed 7/8 of the frames to make the gif
smaller):

![noc_turtle demo spiral](/assets/imgs/noc_turtle/spiral.gif)

The new concept is the SPAWN function, that clones the current turtle and
branches the clone to a new function.  In this case we use it to create a
recursive rule rendering a typical L-tree structure.

# Conclusion

I used the library extensively in my last video game ([Blowfish Rescue], if you
haven't checked it out yet) to create all the backgrounds and most of the game
elements.  A few examples:

Some elements of the game:
![blowfish objects](/assets/imgs/noc_turtle/objs.png)

An animated background:
![blowfish background](/assets/imgs/noc_turtle/city.gif)

For more information about how to use the library, check the file
[noc_turtle.h] in the github repo.  One important aspect is that the library
itself does not do any rendering: instead you pass it a callback function that
will be called each time a turtle need to render something.  That way the code
is totally independent of any rendering library like OpenGL or DirectX.

Do not hesitate to contact me if you want to use noc_turtle in one of your
project and need some tips or info.


[noc_turtle]: https://github.com/guillaumechereau/noc
[Blowfish Rescue]: http://noctua-software.com/blowfish-rescue
[Context Free project]: http://www.contextfreeart.org
[LOGO]: http://en.wikipedia.org/wiki/Logo_%28programming_language%29
[coroutines]: http://www.chiark.greenend.org.uk/~sgtatham/coroutines.html
[noc_turtle.h]: https://github.com/guillaumechereau/noc/blob/master/noc_turtle.h
