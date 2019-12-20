---
layout: post
title: "This Blog is now Hosted on Github"
---

To date, my blog has been hosted on Wordpress. This has irked me for some time ... I'm a software developer and I've always wanted my blog to be powered by something I wrote myself.

Well, it turns out that I don't have time to do that properly. As a half-way house, I've moved everything to [github](http://github.com). It seems like a good compromise as I certainly have more control over the nuts and bolts of the site here, without being mired in Wordpress' arcane PHP code. It also means that the blog is going to be pretty minimalistic ... all the rage these days, isn't it?

Github users can create a new repository called `YOUR-GITHUB-USERNAME.github.com` and it will be accessible at `http://YOUR-GITHUB-USERNAME.github.com`. For example, my repository is sitting at <http://github.com/yeurch/yeurch.github.com> and it's accessible at <http://yeurch.github.com/>. With some CNAME trickery, I can also make it accessible at http://blog.richardfawcett.net, so that's what I've done.

I also took the opportunity to move the blog posts to a new hostname *blog*.richardfawcett.net in order to free up the www. site for other purposes ... eventually some info on my freelancing activities. For now, www.richardfawcett.net will redirect to blog.richardfawcett.net.

Although I'm writing this on 27th March, 2013, I won't switch over to the Github pages until I'm pretty sure that all the redirects from the old permalinks are working well, and that will undoubtedly need me to write a fair bit of custom code on the old site.  Hey, it could easily be 2015 before this is widely available on the web, but I do hope not.

I'm going to sign off with a couple of code snippets, to test that [rainbow.js](http://craig.is/making/rainbows) is doing its thing properly with code samples. This is, after all, a techincal blog.

We'll start with a C# sample ...

<pre data-language="c#">
using System;
namespace Yeurch
{
    public class MyClass
	{
		public static void Main(string[] args)
		{
			Console.WriteLine("Hello, World!");
		}
	}
}
</pre>

And now, something in JavaScript ...

<pre data-language="javascript">
$(function(){
    $('a.someselector').click(function(evt){
		evt.preventDefault();
		console.log('A link was clicked');
	});
});
</pre>