---
layout: post
title:  "Compiling Amiga Specific C (Part 5 of 5)"
---

This is the final part of a five-part series of posts on compiling Amiga-specific C code on an (emulated) Amiga A1200. We’ll start with a machine with an empty hard drive, and over the series, build up to compiling an Amiga-specific program which uses the graphics library.

* Part 1 – [Introduction and motivation](/2021/04/30/compiling-amiga-specific-c-part-1-of-5.html)
* Part 2 – [Installing Workbench and necessary utilities](/2021/05/08/compiling-amiga-specific-c-part-2-of-5.html)
* Part 3 – [Installing VBCC and the Amiga include files](/2021/05/12/compiling-amiga-specific-c-part-3-of-5.html)
* Part 4 – [Compiling a simple Hello World program](/2021/05/19/compiling-amiga-specific-c-part-4-of-5.html)
* **Part 5 – Compiling some Amiga-specific code** (this part)

Let's go!

## Compiling some Amiga-sepcific code

It’s time for some proper Amiga coding! We’ll take some example code from the Amiga Intuition Reference Manual, published by Commodore in 1986, we’ll get it to compile with VBCC. We’ll finish by talking about different compiler options that you may need if you want to go on and make modifications to the program, or write your own killer application.

**The example code**

Here it is. It’s about as straight-forward as it can be. The program creates a screen, opens up a window on that screen, and prints a “Hello Amiga” message in the new Window. It then listens for the user pressing the close button, and shuts down everything cleanly.

I had to make a few modifications from the original as-printed version … it declared some unused variables, and didn’t cleanly close its libraries. Anyway, here it is …

{% highlight c %}
/*****************************************
 * 
 * "Hello Amiga"
 * 
 *****************************************/

#include <exec/types.h>
#include <intuition/intuition.h>

struct IntuitionBase *IntuitionBase;
struct GfxBase       *GfxBase;

#define INTUITION_REV 0
#define GRAPHICS_REV  0

struct TextAttr MyFont = {
    "topaz.font",   /* Font name */
    TOPAZ_SIXTY,    /* Font height */
    FS_NORMAL,      /* Font style */
    FPF_ROMFONT,    /* Preferences */
};

struct NewScreen NewScreen = {
    0,               /* left */
    0,               /* top */
    320,             /* width */
    200,             /* height */
    2,               /* bit depth (4 colors) */
    0, 1,            /* detail pen and block pen */
    NULL,            /* no special display modes */
    CUSTOMSCREEN,    /* screen type */
    &MyFont,         /* use the font we defined above */
    "My Own Screen", /* screen title */
    NULL,            /* no special screen gadgets */
    NULL,            /* no special CustomBitMap */
};

main() {
    struct Screen *Screen;
    struct NewWindow NewWindow;
    struct Window *Window;

    /* Open intuition library */
    IntuitionBase = (struct IntuitionBase*)OpenLibrary("intuition.library", INTUITION_REV);
    if (IntuitionBase == NULL) exit(FALSE);

    GfxBase = (struct GfxBase*)OpenLibrary("graphics.library", GRAPHICS_REV);
    if (GfxBase == NULL) exit(FALSE);

    if ((Screen = (struct Screen*)OpenScreen(&NewScreen)) == NULL) exit(FALSE);

    NewWindow.LeftEdge = 20;
    NewWindow.TopEdge = 20;
    NewWindow.Width = 300;
    NewWindow.Height = 100;
    NewWindow.DetailPen = 0;
    NewWindow.BlockPen = 1;
    NewWindow.Title = "A Simple Window";
    NewWindow.Flags = WINDOWCLOSE | SMART_REFRESH | ACTIVATE | WINDOWSIZING |
            WINDOWDRAG | WINDOWDEPTH | NOCAREREFRESH;
    NewWindow.IDCMPFlags = CLOSEWINDOW;
    NewWindow.Type = CUSTOMSCREEN;
    NewWindow.FirstGadget = NULL;
    NewWindow.CheckMark = NULL;
    NewWindow.Screen = Screen;
    NewWindow.BitMap = NULL;
    NewWindow.MinWidth = 100;
    NewWindow.MinHeight = 25;
    NewWindow.MaxWidth = 640;
    NewWindow.MaxHeight = 200;

    if (( Window = (struct Window*)OpenWindow(&NewWindow)) == NULL) exit(FALSE);

    Move(Window->RPort, 20, 20);
    Text(Window->RPort, "Hello Amiga", 11);

    Wait(1 << Window->UserPort->mp_SigBit);

    CloseWindow(Window);
    CloseScreen(Screen);
    CloseLibrary(GfxBase);
    CloseLibrary(IntuitionBase);

    exit(TRUE);
}
{% endhighlight %}

You *could* type this into `memacs` or `edit`, but that’s going to be really hard work. I’d recommend using a text editor on your host system, and saving the file into the Shared folder that we’ve been using to share files between the host and our Amiga VM. I saved it as `hello_amiga.c`.

**Compiling the program**

The naive approach would be to use something similar to the command we used last time to compile the hello world program:

```
vc -o hello_amiga hello_amiga.c
```

That’s going to produce a bunch of linker errors. Our file compiles just fine, but it makes use of externally declared symbols, like `OpenScreen`. Well, actually it’s `_OpenScreen` … I guess the header files have inline code which adds an underscore and calls an externally defined function of that name. Then the linker comes to tie everything together and produce our executable, it can’t resolve the definition of that routine, so we get linker errors.

vbcc ships with a `amiga.lib` file which contains the implementations of all of the Amiga-specific functions. When we’re compiling a program, we have to tell `vc` that we want to link with that library by adding `-lamiga` to our command line.

```
vc -lamiga -o hello_amiga hello_amiga.c
```

This compiles, and running our executable with `hello_amiga` works as expected …

![Image 1](/assets/images/20210430_amiga/part5.png)

OK, I didn’t say it was _pretty_.

## Some other considerations

**Automatic library opening**

We can remove some of the boiler plate that opens and closes the libraries we need (“intuition.library” and “graphics.library”), as vbcc will handle automatically opening and closing them for us.

Remove the definitions of `IntuitionBase` and `GfxBase` (lines 10 and 11) and all the lines that refer to them.

With the same invocation of `vc` as above, we get a load more linker errors, complaining that `_GfxBase` and `_IntuitionBase` are not defined. We need to link with another magic library that ships with `vbcc`, this one is `auto.lib`. To link against it, we add `-lauto` to the command line …

```
vc -lamiga -lauto -o hello_amiga hello_amiga.c
```

**Floating point math**

At some point, you’re going to want to use some floating point math. E.g. double types and all that good stuff. Well, it doesn’t “just work”. You have to link with yet another vbcc provided library, this time you need to specify:

```
-lmieee.
```

Alternatively, if you don’t need 68000 support, and you can target just 68020 processors or better (i.e. Amiga 1200 or later), you can instead make use of the FPU (floating point unit) with these commands:

```
-lm881 -fpu=68881
```

**C-99 support**

It turns out that vbcc by default supports only a really old version of C. For example, you can’t even use end of line comments prefixed with `//`. You can get C-99 support by adding the following switch to the command-line:

```
-c99
```

## In conclusion

So, that’s about it. We’ve gone from a pristeen Amiga, installed everything we need to do C development, and got some Amiga-specific code to compile and run.

Where you go from here is up to you. I’d highly recommend getting some of the official Amiga reference manuals. There are copies available online, but I’m not sure how legitimate that is from a copyright perspective. Some that you could check out are the “Amiga Intuition Reference Manual”, the “Amiga Hardware Reference Manual”, “Amiga ROM Kernel: Devices” and the “Amiga ROM Kernel Reference Manual: Libraries and Devices”.

Good luck!
