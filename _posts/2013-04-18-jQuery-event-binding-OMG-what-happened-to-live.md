---
layout: post
title: "jQuery Event Binding - OMG What Happened to live()?"
---

One idle Tuesday, I decided to update jQuery from version 1.8.2 to the [latest version, v1.9.1](http://ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.js). I love new shiny things, and the jQuery team do a consistently awesome job of bringing new performance improvements with every release.

So, just change 1.8.2 for 1.9.1 in the CDN's URL, and I'm good to go, right?  NOOOOOOOOOOO!!!!!  They [removed the live() method](http://jquery.com/upgrade-guide/1.9/#live-removed).  This is clearly my fault, and I really need to start [RTFM](http://en.wikipedia.org/wiki/RTFM) before I jump right in.

Still, no harm done. Tests revealed the problem before pushing to production.  You do have tests, right? While fixing the problem, it led me to dig in to the way jQuery handles events, and it was, dear readers, a fascinating journey of discovery.

### TLDR;

Under the hood, jQuery now uses the `on()` method for all event handling. All of the other methods are just convenience methods which wrap `on()`.

<pre data-language="javascript">
$('.myclass').click(fn) ==> $('.myclass').on('click', fn)
$('.myclass').bind('click',fn) ==> $('.myclass').on('click', fn)
$('.myclass').live('click', fn) ==> $(document).on('click','.myclass', fn)
$('.container').delegate('.myclass','click',fn) ==> $('.container').on('click','.myclass',fn)
</pre>

`live()` was removed in version 1.9 of the framework, and there's no reason to suppose that other methods may not be deprecated soon.  Future-proof your code (and gain a slight performance boost in avoiding the wrapper functions) by using `on()` for all jQuery event binding.

### click(), focus() etc.

In the jQuery sourcefile `event-alias.js` on GitHub, you can see that the [click(), focus()](https://github.com/jquery/jquery/blob/562ca75e06a07067cace674e4e492119d78ca161/src/event-alias.js#L1) and such functions are created by splitting a huge string containing all of the event names. If there are no arguments passed, then we call `trigger(eventName)` ([line 8](https://github.com/jquery/jquery/blob/562ca75e06a07067cace674e4e492119d78ca161/src/event-alias.js#L8)):

<pre data-language="javascript">
click()  ==>  trigger('click')
</pre>

Otherwise, if there were arguments passed in, we just pass the function to `on()` ([line 7](https://github.com/jquery/jquery/blob/562ca75e06a07067cace674e4e492119d78ca161/src/event-alias.js#L7)):

<pre data-language="javascript">
click(fn)  ==>  on('click', null, null, fn)
click(data, fn)  ==>  on('click', null, data, fn)
</pre>

### bind()

Again, in `event-alias.js`, you can see at [line 18](https://github.com/jquery/jquery/blob/562ca75e06a07067cace674e4e492119d78ca161/src/event-alias.js#L18) that `bind()` is just a single line wrapper around `on()`. Really simple.

<pre data-language="javascript">
bind(event, data, fn)  ==>  on(event, null, data, fn)
</pre>

### live() and delegate()

Both `live()` and `delegate()` are there to allow event binding on DOM objects that were not present when the page loads ... for example, markup which was injected into the page during an AJAX request. The problem with `live()` is that it always bound itself to the whole document, so any event on the whole page which bubbled up to document level caused the delegate to evaluate whether it should fire.  The `delegate()` method was introduced into jQuery in v1.4.2 as an improvement to `live()`.  When `delegate()` came around, instead of calling `live()` on the selector you wanted events to be triggered by, you call `delegate()` on a container object, which restricts the scope so that it's not being executed for every event on the page.  Instead, the code only runs if an event bubbles up to the container.

The first argument to `delegate()` is a jQuery selector to test the event's sender ... if it matches, the associated function fires.  This allows the delegate call to be bound at page load (against the always present container DOM object), but allows it to check for the existence of elements inside the container at execution time.

In other words, the removed `$('.someclass').live('click',fn)` call is equivalent to 
`$(document).delegate('.someclass', 'click', fn)`.  However, if you know that all instances of `.someclass` will be inside the container `#mycontainer`, then instead, for increased performance, you'd call:

<pre data-language="javascript">
$('#mycontainer').delegate('.someclass', 'click', fn)
</pre>

[Line 25](https://github.com/jquery/jquery/blob/562ca75e06a07067cace674e4e492119d78ca161/src/event-alias.js#L25) of `event-alias.js` shows that `delegate()` just passes straight through to `on()`, but reversing the order of the first two parameters, leading to a more natural ordering, so you'd read "on click" for example.

<pre data-language="javascript">
$(container).delegate(selector, event, fn)  ==>  $(container).on(event, selector, fn)
$(container).delegate(selector, event, data, fn)  ==>  $(container).on(event, selector, data, fn)
</pre>

### on()

A lot of the code of the `on()` method (in [event.js at line 724](https://github.com/jquery/jquery/blob/562ca75e06a07067cace674e4e492119d78ca161/src/event.js#L724)) is used to shuffle arguments around to allow for overloading (lines 741 to 756).  The full signature of `on()` is:

<pre data-language="javascript">
on(event, selector, data, function)
</pre>
[Lines 728 to 739](https://github.com/jquery/jquery/blob/562ca75e06a07067cace674e4e492119d78ca161/src/event.js#L728) handle another overloaded version, where an object is passed, where each key is an event name, and the associated values are functions. E.g.

<pre data-language="javascript">
on({
   click: fn_for_click_handling,
   mouseenter: fn_for_mouse_entering
});
</pre>

### I want to use the latest and greatest, what should I do?

If you're beginning a new project, I'd strongly recommend using the `on()` method for all your data binding. It's efficient, and seems to be the preferred way of doing things at the time of writing.  Using `bind`, `delegate`, and heaven-forbid, `click`, `mouseenter`, `keyup` etc. just adds another layer of indirection, wasting CPU cycles!

With a legacy project, as long as you're not using `live()`, then the old methods will work perfectly fine. Yes, you could go through and change things, but I'm sure there are lots more valuable things you could be doing with your codebase ... correct me if I'm wrong!

Even if you are using `live()`, don't panic, as help is at hand. The __jQuery migrate__ plugin will ensure your old code continues to work. It's available in both [development](http://code.jquery.com/jquery-migrate-1.1.1.js) and [release](http://code.jquery.com/jquery-migrate-1.1.1.min.js) (minified) versions as with most other jQuery plugins. Just include this plugin after you reference jQuery, and you can continue using `on()`.

What's particularly cool is that (with the development version), all use of deprecated or removed methods are logged to the Javascript console. When running it against code which uses `live()`, I get the following logged to the console:

    JQMIGRATE: jQuery.fn.live() is deprecated

Hope this helps!
