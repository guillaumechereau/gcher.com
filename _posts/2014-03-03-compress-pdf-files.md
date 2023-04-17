---
layout: post
title:  Bash script to compress pdf files
subtitle: A useful tool for freelancers
redirect_from: /compress-pdf-files.html
---

Since I work as a freelance, I have to deal with a lot of pdf files I receive
from clients.  Some of them contain uncompressed images that make the files
bigger than they need to be.

On this [stackexchange question][se], a user named Marco gives us a nice way to
use [ghostscript] to compress any pdf file.

There is a loss of quality in the compression, but most of the time it is
acceptable.

Here is the script:


    #!/bin/sh

    INPUTPDF=$1
    OUTPUTPDF=$2
    TMPPDF=$(mktemp)
    METADATA=$(mktemp)

    # save metadata
    pdftk "$INPUTPDF" dump_data_utf8 > "$METADATA"

    # compress
    gs                       \
      -q                     \
      -sOutputFile="$TMPPDF" \
      -sDEVICE=pdfwrite      \
      -dNOPAUSE              \
      -dBATCH                \
      -dPDFSETTINGS=/ebook   \
      "$INPUTPDF"

    # restore metadata
    pdftk "$TMPPDF" update_info_utf8 "$METADATA" output "$OUTPUTPDF"

    # clean up
    rm -f "$TMPPDF" "$METADATA"


[ghostscript]: http://pages.cs.wisc.edu/~ghost
[se]: http://unix.stackexchange.com/questions/50475/how-to-make-ghostscript-not-wipe-pdf-metadata


