---
categories:
  - gtd
  - note
  - text
  - file
  - orgmode
alias: how-to-keep-notes.html
title: Keeping notes in unformatted text files
date: 2016-07-25
---


For years I have been looking for the best way to keep organized notes on my
computer(s).  I tried a lot of different approaches, from web based tools to
emacs [org mode](http://orgmode.org) and desktop wiki.

Now I think I can say that I found the best way (for my personal case) to do
it: I keep all my notes as unformatted text files, in a single flat directory.

Here is how it works: I have a directory `Notes` in my computer home folder.
Every time I need to take a note I put it in a text file named `<TOPIC>.txt`
inside this folder.  Even though I usually use something that looks like
markdown, I do not care about the format of the file, just plain unformatted
text.

So why does this method sticks when all the other failed?  Let see.

##1 Web based wiki

This was my first approach: I was keeping all my notes in a personal private
wiki.  One advantage is that I could store images.  The big problem was that
it was too slow to open the wiki every time I need to make a small change to
a note (pretty much all the time).

##2 org mode

After I stopped using a wiki, I turned to [org mode](http://orgmode.org).  I
have to say that org mode is actually quite good, and in fact very close to my
current approach.  I had two problems with org mode: the first is that I spent
too much time making sure my org mode files were correctly formatted, and the
second is that I stopped using emacs (originally to help with carpal tunnel
syndrome).

##3 zim (or other desktop wikis)

[Zim](http://zim-wiki.org/) seemed like a good choice for a while, because I
could still edit my notes with any text editor, but I also had the ability to
browse them like a wiki with links and formatting.  The problem was that, just
like with org mode, I found it too troublesome to maintain the formatting,
no matter how simple it was.

##Conclusion

I have been keeping my note as unformatted text files for years now, I and
really got used to it.  The cool thing about it is that since by design I don't
use any formatting, I can quickly add to a note without triggering my computer
OCD and worrying about it.  If for some reason I want to use some structure in
a note, I just do it on a need basis.

For example some of my note files have special meaning:

- `tmp.txt` is used to keep short term notes, things that I know I
   won't use after a day or two.
- `todo.txt` is just a list of things I have to do.
- `days.txt` is a simple diary of my work time.  I use a script to
   automatically parse it for charging my clients.

Since the notes are in plain text, I cannot have links between them, but in
practice I find it much easier to use `grep` to quickly find all the notes with
a specific word in it.
