---
categories:
  - gtd
  - reming
  - productivity
aliases: personal-calendar-using-remind.html
subtitle: How I keep organised
title: My personal calendar using Remind
date: 2014-02-24
---


I made a cool _"getting things done"_ script on my computer: any time I type
the 'rem1' command, I see a nicely formatted calendar of the current and
following three weeks.  It also prints out the most urgent things I need to do.
It looks like that:

![remind output](/assets/imgs/rem1.png)


In the past I tried several different note tracking tools, most notably the
great [org-mode].  I like the concept of keeping all my personal notes locally
into plain text files.  The problem I have with org-mode is that it only works
with emacs, while I prefer to use vim.

Following the unix philosophy, I think the most important is not the tools used
to process the data, but the way we store them, and plain text is obviously
superior to anything (note: xml is _not_ plain text!).  For example even though
I don't even use emacs anymore, I can still open and read my old org-mode
files, and I know I always will.

All my notes file are stored locally on my computer in the directory *~/Notes*.
Some of the files have special meaning, for example ~/Notes/todo.txt contains
my to-do list.  I also have one file per day for my diary and schedule, stored
as *~/Notes/days/yyyy-mm-dd.txt*.

Now, I wanted a way to print out an overview of my daily and weekly schedule.
That's where [remind] comes to play.  Remind is a unix tool similar to the
venerable 'calendar' command.  I am not going to enter into the details of
remind, it is a flexible software used to define calendar events in text files,
and generate different kind of outputs from it.  The syntax of a basic remind
event is:

    REM 2014-02-24 write a new blog post

It can get much more complex than that, for an comprehensive documentation
refer to the [man page][remind-man].

Of course I do not want to change the format of my note files to accommodate
any special tool.  Instead what I did is write a small python script that
parses my files, generate the corresponding remind file, and then invoke remind
on it to show me the nice output you can see in the beginning of this post.

(note: I am not distributing the script for the moment, since it is dependent
on the way I organize my note files, but if you have some interest to see it,
please drop me an email, I'll make it available under a GPL licence.)

I am quite satisfied with the result.  I think many people do not want to use
remind because of the forced syntax, but once you use it as an intermediary
step in the process of managing your note files it can come in handy.


[org-mode]: http://orgmode.org
[remind]: http://www.roaringpenguin.com/products/remind
[remind-man]: http://linux.die.net/man/1/remind
