---
layout: post
title:  Vocabulary learning wallpaper with python and gnome
subtitle: Show Chinese words on your gnome desktop
categories: chinese productivity gnome linux
redirect_from: /chinese-wallpaper.html
---

Here is a simple way to customize your gnome desktop to show random text taken
from a file.  I use it to learn Chinese, but it can be easily modified for any
other kind of textual information.

This is how I did it:

1. Using inkscape, I made a template SVG file for the background.  I started
   from the default gnome 3 wallpaper, and I added three text elements: one for
   the Chinese characters, one for the pinyin, and one for the English
   translation.  In place of the text I used `"@chinese@"`, `"@pinyin@"`, and
   `"@english@"`.  Later my script will substitute those placeholder words with
   the correct values.

2. I created a input file with all the entries I want to learn.  The file looks
   like this:

        容易 [rong2 yi4] easy
        決心 [jue2 xin1] determination
        期待 [qi1 dai] to look forward
        冷淡 [leng3 dan4] cold, indifferent
        寂寞 [ji4 mo4] lonely
        突然 [tu1 ran2] sudden
        反正 [fan3 zheng] anyway
        招牌 [zhao1 pai] signboard
        討厭 [tao3 yan4] to dislike
        努力 [nu3 li4] great effort, to strive
        原諒 [yuan2 liang4] to excuse, to forgive, to pardon

3. I wrote this python script.  Every 10 minutes, the script would pick a new
   random entry from my input file, generate a png file from the svg template,
   and run the gsettings command to use it as a new wallpaper.

    #!/usr/bin/env python
    
    import codecs
    import os
    import random
    import re
    import shutil
    import subprocess
    import tempfile
    import time
    
    
    INPUT_FILE = "/home/guillaume/notes/chinese.txt"
    PERIOD = 60 * 10        # Change image every 10 minutes
    PWD = os.path.dirname(__file__)
    
    
    def get_entry():
        entries = []
        reg = re.compile(r"^(.+?) \[(.+)\] (.+)$")
        for line in codecs.open(INPUT_FILE, "r", "utf-8"):
            m = reg.match(line)
            if m is None:
                continue
            entries.append((m.group(1), m.group(2), m.group(3)))
        return random.sample(entries, 1)[0]
    
    
    def create_png(tmpdir, chinese, pinyin, english):
        svg = open("%s/wallpaper.svg" % PWD).read()
        svg = svg.replace("@chinese@", chinese)
        svg = svg.replace("@pinyin@", pinyin)
        svg = svg.replace("@english@", english)
        svg_path = "%s/wallpaper.svg" % tmpdir
        png_path = "%s/wallpaper.png" % tmpdir
        out = codecs.open(svg_path, "w", "utf-8")
        out.write(svg)
        out.close()
        subprocess.call(["inkscape", svg_path, "-e", png_path])
        return png_path
    
    
    def set_wallpaper(path):
        subprocess.call(["gsettings", "set",
                         "org.gnome.desktop.background", "picture-uri",
                         "file:///%s" % path])
        subprocess.call(["gsettings", "set",
                         "org.gnome.desktop.background",
                         "picture-options", "zoom"])
    
    
    while True:
        tmpdir = tempfile.mkdtemp("chinese-wallpaper")
        entry = get_entry()
        png_path = create_png(tmpdir, *entry)
        set_wallpaper(png_path)
        time.sleep(PERIOD)
        shutil.rmtree(tmpdir)

4. I added the script to my startup application (using
   *gnome-session-properties*).


5. Log out and log in again.  Now my wallpaper looks something like this:

![Chinese wallpaper](/assets/imgs/chinese-wallpaper.png)

