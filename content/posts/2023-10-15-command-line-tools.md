---
categories: [linux, terminal]
title: Some command line tools I use
date: 2023-10-15
---

In this post I am going to present a few of my favorite command line tools
that might be of interest to programmers who prefer to work directly from the
terminal.

(Note: since I didn't want to put screenshots or html, I removed the
colors from the examples.  Most of those tools actually make use of
ANSI escaping to colorize their outputs)


## [ag] -- The Silver Searcher

```plaintext
$ ag LookAdjust
src/g_game.cpp
797:static int LookAdjust(int look)
823:    look = LookAdjust(look);
865:    yaw = LookAdjust(yaw);
```

This is like `find` and `grep` merged together into a simpler command.
Specially useful when searching large code base.

What I like about it is that it is extremely fast, as far as I can tell as
fast as the file system allows it.  It also automatically skips files from
.gitignore, so it is prefect to quickly look for some functions or text in a
project.


## [tig] -- Text mode interface for Git

```plaintext
2023-08-24 20:40 +0800 Guillaume    o [master] {origin/master} {origin/HEAD} Attempt at fixing windows build since last commit
2023-08-23 16:02 +0800 Guillaume    o Fix security issue in quickjs-libc.c
2023-07-24 18:11 +0800 Guillaume    o Fix compilation on OSX
2023-07-23 23:13 +0800 Guillaume    o Add an option to simplify exported glTF
2023-07-23 19:43 +0800 Guillaume    o Optimize exported gltf mesh with meshoptimizer
```

You can use this as a replacement on any `git` log/show/diff/blame command and
you can then explore the output in a nice ncurses interface.  It supports the
vim movement shortcut, pressing enter on any commit opens the diff in a
separate view.

By default, just typing 'tig' in a project will shows the commit logs

This is the equivalent of a fancy git graphical interface, but without
losing the advantages of the command line.


## [aunpack] -- Decompress archives of any type

An invaluable tool to decompress zip or any other archive files.
The best feature is that it automatically detects if the archive doesn't have
a root directory and creates one before unpacking if needed, preventing
the annoyance of extracting many files in the wrong directory.
Who remembers all the arguments needed when calling `unzip` or `tar`?

```plaintext
$ aunpack blenderkit-v3.4.0.230122.zip
Archive:  blenderkit-v3.4.0.230122.zip
   creating: Unpack-8063/blenderkit/
  inflating: Unpack-8063/blenderkit/__init__.py
  inflating: Unpack-8063/blenderkit/thumbnails/vs_rejected.png
  inflating: Unpack-8063/blenderkit/thumbnails/vs_uploaded.png
  inflating: Unpack-8063/blenderkit/thumbnails/vs_uploading.png
  inflating: Unpack-8063/blenderkit/thumbnails/vs_validated.png
  ...
blenderkit-v3.4.0.230122.zip: extracted to `blenderkit'
```


## [vifm] -- Vi file manager

This is similar to the famous `midnight commander`(mc) terminal file manager.
For some reason I never liked `mc` that much.  I usually only want to
browse my file system and copy files from one directory to another, and
since I am a vim user, vifm does the job very well.  I would not recommend
it to non vim users though, since all the shortcuts are the same as in vim.

```plaintext
 ~/Projects/goxel                                 ~/Projects/goxel/src
 *../                 4 K                          ../                 4 K
  android/            4 K                          assets/             4 K
  data/               4 K                          formats/            4 K
  doc/                4 K                          gui/                4 K
  ext_src/            4 K                          mobile/             4 K
  osx/                4 K                          tools/              4 K
  screenshots/        4 K                          utils/              4 K
  snap/               4 K                          action.c          2.5 K
  src/                4 K                          action.h          2.9 K
  svg/                4 K                          action.o           60 K
  tools/              4 K                          actions.h         2.3 K
  AUTHORS           482 B                          assets.c            2 K
  CHANGELOG.md      2.3 K                          assets.h          1.2 K
  CONTRIBUTING.md   2.6 K                          assets.o          293 K
  COPYING            34 K                          block_def.h       3.6 K
  INTERNALS.md      1.3 K                          box_edit.c        5.7 K
  Makefile          1.8 K                          box_edit.o        458 K
  README.md           3 K                          camera.c          5.4 K
  SConstruct        4.7 K                          camera.h          3.5 K
  config.log        1.3 K                          camera.o          410 K
  goxel              67 M                          config.h          1.4 K
  icon.png          116 K                          dialogs_osx.m     928 B
  todo              663 B                          file_format.c     2.3 K
                                                   file_format.h       2 K
```


# [ledger] -- Command-line, double-entry account reporting tool

Ledger is not just a command line tool, but mostly a text format to do
accounting.  Keeping my finance in plain text files has a lot of advantages
for me since as a programmer I already have many tools to manage text files.
Ledger itself is not the simplest of the command from this list, but there is
`hledger` which provides a convenient ncurses interface to browse the data.


## [ncdu] -- NCurses Disk Usage

```plaintext
ncdu 1.18 ~ Use the arrow keys to navigate, press ? for help
--- /home/guillaume/Projects/goxel -------------------------
  778.6 MiB [######################] /android
   90.3 MiB [##                    ] /src
   67.3 MiB [#                     ]  goxel
   59.4 MiB [#                     ] /.git
   15.9 MiB [                      ] /ext_src
    1.3 MiB [                      ]  .sconsign.dblite
  432.0 KiB [                      ] /screenshots
  420.0 KiB [                      ] /data
  412.0 KiB [                      ] /osx
  132.0 KiB [                      ] /doc
  128.0 KiB [                      ] /svg
```

It seems that my computer is always low in disk space.  Even though
ubuntu graphical disk usage analyzer is quite good, sometimes it's nice
to be able to check my disk space from command line.  Ncdu is another
ncurses interface that does the job well and get out of the way.


## pdf2txt -- extracts text contents of PDF files

I sometimes use this tool when doing my company accounting, since until
recently my bank only provided report in pdf format.


## "python3 -m http.server" -- serve local files on a web server

Not a tool er se, but a useful command that is worth memorizing.  I use it for
example if I want to quickly copy a file from my PC to my iPad.


[ag]: https://geoff.greer.fm/ag/
[tig]: http://jonas.github.io/tig/
[aunpack]: https://linux.die.net/man/1/atool
[vifm]: https://vifm.info/
[ledger]: https://plaintextaccounting.org/
[ncdu]: https://dev.yorhel.nl/ncdu
