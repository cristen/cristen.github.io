---
layout: post
title:  "Plain text file to PDF : How to ?"
date: 2016-07-27 06:30:52 GMT+1
categories: Utilities
---

# Foreword

Before you blindly type those commands below, you’re going to need a few tools first. They can be installed on Archlinux like this : 

{% highlight bash %}pacman -S ghostscript enscript{% endhighlight %}

(iconv is already in the core packages but if, for whatever reason, you need it, you’ll find it inside the glibc package)

# One CLI to rule them all
The command line interface is an amazing tool and you can perform a lot of stuff with just a few commands. Today I’m gonna show you how to convert a text file to a PostScript file and then convert it to PDF.

# (Optional) Encoding conversion
To convert the file from one encoding to another we’re going to need the **“iconv”** utility.

{% highlight bash %}
iconv -f LATIN1 -t UTF8//TRANSLIT my_text_file.txt -o my_converted_text_file.txt
{% endhighlight %}

The *-f* is the input encoding and the *-t* the output encoding. You can append the *“//TRANSLIT”* thingy so that characters that cannot be represented in the target will be approximated with a similar character.

# Create the PostScript file
A postscript file is a file that is interpreted by printers, it’s the ancestor of the PDF format. Made simple, it describes everything about your document to the printer using a specific language called “PostScript”. Instead of sending it to the printer, we’re going to keep it on the computer and convert it later to PDF.
{% highlight bash %}
enscript -b "" -p printable.ps my_converted_text_file.txt
{% endhighlight %}
So here the *-b* option sets the header to an empty value (because the header is printed on every page).

As for the rest, it is self explanatory.

# PostScript to PDF
Ghostscript is an interpreter for both PostScript and PDF. This last step converts the PostScript file to PDF using ghostscript.
{% highlight bash %}
ps2pdf printable.ps final.pdf
{% endhighlight %}

And tadaaa, your file is ready !

I hope it helped !
