---
categories:
  - linus
  - subsurface
  - joke
  - git
  - code
aliases: /linus-subsurface.html
subtitle: Linus Torvald is a funny guy
title: Linus subsurface funny commits
date: 2014-01-13
---


Linus Torvald is mostly famous for being the creator of the [Linux
Kernel][linux] and [git][git].  He is also the author of a diving application:
[subsurface][subsurface].

[linux]: https://www.kernel.org/
[git]: http://git-scm.com
[subsurface]: http://subsurface.hohndel.org/

What I like about subsurface is that we can see how Linus works on a small
scale project, similar to the kind of things most developers like me work on.

I really recommend reading the [git logs of Linus][logs]: not only are they
perfectly written and formatted (check how he separates sentences with two
spaces), but they also sometime contain funny parts.  Here are a few examples
of what we can find in it:

[logs]: http://git.hohndel.org/?p=subsurface.git;a=search;s=Linus+Torvalds;st=author


    Gaah.  XML is *stupid*.  It's not easy to parse for humans or for
    computers, and some of these XML files are just disgusting.

----

    I need to get stinking drunk before I look at more xml mess.

----

    .. nice compiler warning hidden by the crazy gcc pointer sign warnings
    that nobody wants to see (yes, we really do want to do 'strlen()' even
    on unsigned strings, don't complain, crazy bitch compiler).

----

    "But XML is portable". Crazy people.

----

    I'd like to ask what drugs Suunto people are on, but hey, it's a Finnish
    company.  No need to ask.  Vodka explains everything.  LOTS AND LOTS OF
    VODKA.
    In comparison, the libdivecomputer output is nice and sane

----

    Did I just say "In comparison, the libdivecomputer output is nice and
    sane"?

    It turns out that libdivecomputer has been doing some drugs too when it
    comes to gas mixes.  Like showing O2 percentages as 255.0% and N2
    percentages as -155.0%.

----

    "subsurface"? I don't know.  Maybe I should follow the "name all
    projects after myself" mantra.  "divenut"?

----

    Oh Gods. Why are all other scuba programs so f*&% messed up?

----

    Ok, this is the ugliest f*&$ing printout I have ever seen in my life,
    but think of it as a "the concept of printing works" commit, and you'll
    be able to hold your lunch down and not gouge out your eyeballs with a
    spoon.  Maybe.

----

    Make the printout look different
    
    Not *better* mint you. Just different.
    
    I suck at graphs.

----

    Sue me: I'm not a fan of Serif

----

    xml-parsing: accept 'sitelat' and 'sitelon' for GPS coordinates
    
    Are they ugly and insane tags? Yes.  Are they used? Bingo.  MacDive uses
    this lovely format for specifying dive site location.

    Christ, if you look up "Ugly dialog" on Wikipedia, I think it has a
    picture of this "New dive" thing.  Or it should have.

----

    Make our 'ascii_strtod()' helper more generic
    
    We'll want to do sane parsing of strings, but the C library makes it
    hard to handle user input sanely and the Qt toDouble() function
    interface was designed by a retarded chipmunk.
