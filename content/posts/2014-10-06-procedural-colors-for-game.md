---
categories:
  - code
  - graphics
  - games
aliases: /procedural-colors-for-game.html
commentsUrl: http://blog.noctua-software.com/procedural-colors-for-game.html
title: Procedural colors for video games
date: 2014-10-06
---


Recently I have been spending a lot of time thinking about how to use
procedural colors in my -still unnamed- incoming video game.

[**edit**]: The game is done now, it is called [Blowfish Rescue], and you
can play it for free on Android of iOS.

This is a problem that I am sure many other indie game developers have been
thinking about, and there are not so many resources about it online, so I
though I would share my experience so far on the subject.

# The problem

The question I asked myself is the following one: from a random seed value, how
to generate a set of RGB colors to assign to the elements of my game, such that
the final rendering will look reasonably good.

The algorithm should also allow to specify some extra constraints for certain
element color (for example I want the sky to alway look more or less blue).

And of course, I want to get as much variation as possible in the generated
sets of colors.

# HSV color space

The first thing I realized is that the RGB color representation used by most
graphics API is not well suited for this kind of problem.  Humans perceive
colors in term of hue, value, and saturation or chroma.  So just like in many
mathematical problems, the solution comes from choosing a better referential.

My first choice was to use the HSV color space, since it is very easy to
convert back and forth between RGB and HSV.  Also gimp color selector allows to
enter values in HSV, making it easier to make tests.

However I soon realized that HSV actually match rather poorly our actual
perception of Hue Saturation and Value.  That is, changing the hue component of
the HSV representation of a color will also change it perceived lightness and
saturation.  At first I though that it wouldn't be a problem for my algorithm,
but in fact it makes a substantial difference.  For example here is a screen
shot of the chapter menu of my game where each button color has a fixed HSV
saturation and value, and the hue is uniformly incremented:

![Menu using HSV colors](/imgs/procedural-colors/menu-hsl.png)

We can see that the red button looks darker than the other buttons.

Here is the same menu, using this time the LCH color representation with fixed
chroma and lightness:

![Menu using LCH colors](/imgs/procedural-colors/menu-lch.png)


# LCH color space

HSV (or the related HSL) are not really suitable for actual representation of
color components in term of hue, lightness and saturation.  So instead I used
the more advanced CIE LCH color space (Light, Chroma, Hue).

LCH is based on the CIE Lab color space and closely match our perception of
hue, light and chroma.  The algorithm to convert from RGB to LCH and reverse
can be found on the [great website of Bruce Lindbloom][brucelindbloom].

[brucelindbloom]: http://www.brucelindbloom.com/

Let's compare two color wheels of HSV and LCH:

![HSV color whee](/imgs/procedural-colors/color-wheel-hsv.png)

![LCH color whee](/imgs/procedural-colors/color-wheel-lch.png)

The colors in the LCH wheel look more uniform.  You might think that they also
look a bit boring, but this is in fact the feeling you would expect when all
the colors differ only by their hue.

Now, the problem with the LCH color space is that many LCH values don't map to
a RGB color.  For example here is the LCH color wheel with different Lightness
values

![LCH color wheel holes](
    /imgs/procedural-colors/color-wheel-lch-hole.png)

Some colors are missing, specially when the lightness value gets to close to 0
or 100.

Since this might be a problem for our algorithm, and I don't need to get
perfect values, I decided to fill the gap using an interpolation between the
closest valid RGB value and the HSL value.

![LCH color wheel holes fixed](
    /imgs/procedural-colors/color-wheel-lch-hole-fixed.png)

This is not perfect, but will do for the purpose of my algorithm.


# Color harmony

Many resources online about color theory mention the color harmonies rules.
The idea is that some colors will look better placed next to each other if
their hue are placed at specified relative position within the color wheel.

What few resources point out is that the implicit color wheel used for this is
the paint mixing color wheel, where blue is the opposite to orange, and green
is opposite to red.

This is not really the case in my LCH color wheel, so I deformed the hue values
of my color wheel to make it look closer to the paint mixing color wheel.

![LCH color deformed](
    /imgs/procedural-colors/color-wheel-lch-hole-fixed-hue-deformed.png)

And this is my final color wheel, so to summarized, I started from the default
LCH color wheel, fixed the wholes using HSL values, and deformed the hue to
match more closely to the color mixing wheel.  Here are a few examples for
different chroma values:

![some color wheels](/imgs/procedural-colors/color-wheel-smalls.png)

# Color distance

My color generation algorithm will use the notion of color distance (either to
force a color to be close to a referential color, either to force two colors
not to look too close).  The formula to compute LCH color distance can also be
found on Bruce Lindbloom website.


# The algorithm

And finally, here is the algorithm I use so far:

    1) for each element of the game (sky, walls, walls iner part,
       fish, fluid, etc.) I predefine some optional constraints:

        - lightness range (usually just a fixed value)
        - chroma range    (also usually a fixed value)
        - color that we should be close to
        - color that we should be far from

        This allow for example to force the sky to be blue,
        or the fans color to be different from the walls color.

    2) pick randomly a color harmony type (complementary,
       split complementary, triad, tetradic or square).

    3) pick randomly a base hue value.

    4) for each game element:

        a) randomly pick a color within the color harmony.

        b) if the color does not fit the constraint return to 'a'


# results

I might still try to change the algorithm in order to get more color variation,
or to simplify the color wheel generation, but the results I got so far are not
that bad, at least none of the level look too ugly.

Here are some screen shots of different levels of the game:

![screenshot 00](/imgs/procedural-colors/screenshot-00.png)
![screenshot 01](/imgs/procedural-colors/screenshot-01.png)
![screenshot 02](/imgs/procedural-colors/screenshot-02.png)
![screenshot 03](/imgs/procedural-colors/screenshot-03.png)
![screenshot 04](/imgs/procedural-colors/screenshot-04.png)
![screenshot 05](/imgs/procedural-colors/screenshot-05.png)
![screenshot 06](/imgs/procedural-colors/screenshot-06.png)
![screenshot 07](/imgs/procedural-colors/screenshot-07.png)
![screenshot 08](/imgs/procedural-colors/screenshot-08.png)
![screenshot 09](/imgs/procedural-colors/screenshot-09.png)
![screenshot 10](/imgs/procedural-colors/screenshot-10.png)
![screenshot 11](/imgs/procedural-colors/screenshot-11.png)



# Additional resources

Here are a few link to resources that I found useful when developing my
algorithm:

- [Bruce Lindbloom website][brucelindbloom]: All the algorithm to convert
  between many color space.
- [Devmag][devmag]: Interesting blog post covering some other ideas to
  procedurally generate colors.
- [tigercolor][tigercolor]: description of the different color harmonies.
- [ryb pdf][ryb]: paper explaining how to convert from RYB to RGB.

[Blowfish Rescue]: http://noctua-software.com/blowfish-rescue/get
[brucelindbloom]: http://www.brucelindbloom.com/
[devmag]: http://devmag.org.za/2012/07/29/how-to-choose-colours-procedurally-algorithms/
[tigercolor]: http://www.tigercolor.com/color-lab/color-theory/color-harmonies.htm
[ryb]: http://threekings.tk/mirror/ryb_TR.pdf
