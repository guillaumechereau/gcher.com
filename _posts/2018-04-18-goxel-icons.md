---
layout: post
title: Creating application icons with inkscape
categories: goxel voxel design inkscape icons
redirect_from: /goxel-icons.html
---

Yesterday I tried to improve the style of my voxel editor [goxel].  One thing
I wanted to do in particual is have better looking icons in the UI.

The current icons looked like that on my non retina screen:

![goxel before](/assets/imgs/goxel-icons/orig.png)

This is what I came up with:

![goxel after](/assets/imgs/goxel-icons/curr.png)

It's interesting to work on art for computer screen because for such small size
images, the pixels aliasing quality becomes primordial.  For example in goxel
on a non retina screen, the icons should all have a size of 22x22 pixels.  On a
double density screen, they are 44x44.

When you draw the icons (I use inkscape)
you need to keep in mind the target size, since the rasterisation will make
anything not pixel aligned blurry.

Here is an example of an arrow icon, and what it looks like exported as a
44x44 png image, and then resized to 22x22 using:

![arrow before](/assets/imgs/goxel-icons/arrow.png)

The 22x22 version doesn't look sharp, because the interpolation used when
resizing blurred the separation between the lines.  In order to prevent this
we have to make sure that the source image aligns properly with the final
22x22 image pixel position:

![arrow before](/assets/imgs/goxel-icons/arrow2.png)

Unfortunately inkscape doesn't have live preview of rastersized output, so
it takes some tests to design proper looking icons.  The trick (that I saw
in blender) is to set up the inkscape grid as a fraction of a pixel, with
one major grid line every pixel, that way it is easy to draw lines exactly
centered into a given pixel.  I also use lines thinner than a pixel to make
the result look a bit softer:

![add icon](/assets/imgs/goxel-icons/add.png)

![add icon](/assets/imgs/goxel-icons/add2.png)

[goxel]: https://guillaumechereau.github.io/goxel/
