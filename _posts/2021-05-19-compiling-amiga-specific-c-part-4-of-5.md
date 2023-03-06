---
layout: post
title:  "Compiling Amiga Specific C (Part 4 of 5)"
---

This is part four of a five-part series of posts on compiling Amiga-specific C code on an (emulated) Amiga A1200. We’ll start with a machine with an empty hard drive, and over the series, build up to compiling an Amiga-specific program which uses the graphics library.

* Part 1 – [Introduction and motivation](/2021/04/30/compiling-amiga-specific-c-part-1-of-5.html)
* Part 2 – [Installing Workbench and necessary utilities](/2021/05/08/compiling-amiga-specific-c-part-2-of-5.html)
* Part 3 – [Installing VBCC and the Amiga include files](/2021/05/12/compiling-amiga-specific-c-part-3-of-5.html)
* **Part 4 – Compiling a simple Hello World program** (this part)
* Part 5 – [Compiling some Amiga-specific code](/2021/05/28/compiling-amiga-specific-c-part-5-of-5.html)

Let's go!

## Compiling a simple Hello World program

This one should be a pretty simple one. We’re going to compile a simple Hello World program to basically ensure that we installed our compiler and include files correctly.

So, here’s the simple C program we’re going to be compiling. It’s very straight-forward:

{% highlight c %}
#include <stdio.h>

int main(int argc, char** argv) {
    printf("Hello World, from the Amiga.\n");
    return 0;
}
{% endhighlight %}

AmigaOS comes complete with a text editor called memacs. While primitive by today’s standards, it was a great program for its time. Let’s fire it up by opening a Shell, changing to the `RAM:` directory, and editing a new file we’ll name `hello.c`.

![Image 1](/assets/images/20210430_amiga/part4-01.png)

Type in our program, before choosing “Save-Exit” from the Project menu.

![Image 2](/assets/images/20210430_amiga/part4-02.png)

The C compiler that comes as part of VBCC is actually called vc, and we want to specify an output file of hello by using the -o parameter.

{% highlight plaintext %}
3.Ram Disk:> vc hello.c -o hello
{% endhighlight %}

Assuming we didn’t get any errors, we can then run the program by executing `hello`, like this:

![Image 3](/assets/images/20210430_amiga/part4-03.png)

That’s really all there is to it. If you got errors when compiling, it’s worth checking `S:user-startup` to ensure that it contains the VBCC stuff below (the installer should have automatically added this).

{% highlight plaintext %}
;BEGIN vbcc
assign >NIL: vbcc: System:c_dev/vbcc
assign >NIL: C: vbcc:bin ADD
setenv VBCC vbcc:
;END vbcc
;BEGIN vbcc-m68k-amigaos
assign >NIL: vincludeos3: vbcc:targets/m68k-amigaos/include
assign >NIL: vincludeos3: "System:c_dev/NDK_3.9/Include/include_h" add
assign >NIL: vlibos3: vbcc:targets/m68k-amigaos/lib
;END vbcc-m68k-amigaos
{% endhighlight %}

So, to summarize, we’ve verified that the compiler is installed and working correctly, and can locate and use the standard include files. Next time, we’ll move on to our actual goal of compiling some Amiga-specific code!
