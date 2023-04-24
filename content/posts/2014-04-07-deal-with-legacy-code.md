---
categories:
  - software
  - opinion
alias: /deal-with-legacy-code.html
subtitle: Something we all hate, but all have to do at some point
title: How to deal with spaghetti legacy code
date: 2014-04-07
---


At least three times in my career as a programmer, I had to work on huge,
complicated code written by other people.  In one case I was specifically
hired to replace the original maintainer who -as I later learned- left the
company from a burnout caused by the frustration of having to maintain his own
code.

Inheriting legacy code is common, and few people enjoy it.  Here I put down a
few ideas that I think might help to deal with the situation.


## Don't blame the original coder

It is easy to implement an algorithm using complicated code, everybody can do
it.  The difficulty is to implement the same algorithm using simple code.  That
is what only few people can achieve.  It is hard mostly because for the person
who writes the code, it always seems simple.  It already happened to me that I
had to fix a bug in some old software, wondering who was responsible for such a
bad code, before realizing I was the original author.  Beside, the quality of
simplicity is different for everybody.  Some people will find C++ classes
difficult to read, while some other might have troubles with C pointers.  When
you have to work on a project, it is best to adjust to the internal logic of it
rather than dismissing all the code as ugly.

Always remember that the guy coming after you might well say that *you* are a
bad coder.



## Get the full source code history in a local git repository

If you are lucky enough, the project would already use git (or an other
distributed source code management system), but most of the time you will have
at best a SVN server, and at worse a few zip files containing snapshots of the
code at different time.  If you can, convert the project into a git repository
for your personal use.  There are various tools that can help you to do that.
The reason why you need that is because as you will try to understand the code,
you will need to guess not only what it does, but also _why_ it does it.  To
answer this, being able to dig into the history of the code is invaluable.  For
example you might see some tricky lines of code that do not seem to be needed.
With the `git blame` command, you can see at what revision this code was added.
Maybe this code wasn't there before, and was added to solve a specific issue
that you might not understand yet.

Also, the code probably started simple, and did grow into the spaghetti mess
that you have to deal with.  Reading the first versions of the code will allow
you to differentiate the actual logic of the code from the hacks that have
grown around it.


## Make sure build and deploy works

This is the first thing to do, even before you start to read the code.  You
should know how to compile it and run it.  It sounds obvious, but often, poorly
maintained projects have complicated build process, you might need to download
some extra libraries, to manually copy files around at each compilation, etc.
the compilation of the project shouldn't need anything else than calling
`make`, and it is a waste of time to try to improve the code while this is not
the case.


## Learn how to navigate the code

You are going to spend most of your time reading code, instead of writing it.
Code reads a bit like those "book-where-you-are-the-hero", in the sense that
you keep jumping from function to function, possibly in different files.  If
you use vim or emacs, check ctags or the several code browsing plugins.  If the
code has been written with a specific IDE, it might be better to just use it,
even if this is not you favorite one.  Spending time greping in files is not
fun and will frustrate you too much, so make sure you have the proper
environment that allows you read the code as quickly as possible.


## Refrain from fixing 'anti-pattern' (just yet)

We have all been taught that global variables are bad, that we should use
classes to abstract specialized algorithms, that functions and variable names
should have meaningful names, that we should use accessors to read and write
instances attributes, etc.  Every programmer has his own list of best practices
that he tries to follow.  So of course, when we read other people code, we get
upset to encounter those "anti-patterns" all around the code, and our first
reaction is to try to remove them.  This is a bad idea.

First of all, you shouldn't re-factor the code before you have a deep
understanding of it.  Many seemingly minor changes could have side effects that
you will not be able to track.  So the rule should be to try to touch the code
as little as possible in the beginning.

Then, those anti-pattern could be there for a good reason.  Sometime it makes
sense no to follow the rules, specially in big complicated projects.  You might
spent hours trying to make the code follow your personal idea of cleanliness
just to realize that it is not possible in that case.  Instead, when you see
something that seems just plain ugly, dig into the code history and try to
figure out why the code is like this.

Finally, I think that many of those so called "best practice" are in fact
stupid rules that people shouldn't follow blindly.  The only rule that really
matters is that the code should be easy to read, and should have as little
abstraction layers as possible.


## Ignore comments

In my experience, average quality code are full of comments, and most of them
are only noise on the screen.  Even when you see a comment that actually
provides some extra information that the code does not carry, you should still
be suspicious: the comments might be outdated or just plain wrong.  It is
better to just read the code and understand it, and treat comments as hints.


## Start with small bug fixes

It's going to take some time before you understand the code enough to clean it.
So at the beginning, it is more rewarding to focus on small parts of the code,
one at a time.  The best is to try to fix specific bugs that you can reproduce.
That way you will build up your knowledge of the design of the code.  After a
few weeks of working with the code, you will start to get a deeper
understanding of it, some of the patterns will become clear, and you will be
able to "stop seeing the code", but directly the algorithms.  Once you have
reached this point, you can start to work on bigger tasks.  Trying to fix
everything too quickly is just asking to get burned out.


## Re-factor only if it makes the code smaller

After you've been working on the code for a while, you can try to actually
re-factor the code to make it easier to read and maintain.  I think a good rule
to keep in mind is that a re-factoring is only an improvement if it fixes an
actual bug, or if it makes the code smaller.  Spaghetti code usually contains a
small ratio of business code / total code.  One possible re-factorization could
be to use a well known library to replace part of the code, or to remove unused
functions (poorly maintained projects are full of it).
