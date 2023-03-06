---
layout: post
title:  "Capturing Mutable Variables is Bad"
---

I once got caught out by a really hard to spot bug in C#. I was automatically capturing a variable in a lambda, and that variable was mutable … it was a value defined in a for loop. I’ve created the simplest possible repro to illustrate the problem:

{% highlight csharp %}
class Program {
	static void Main(string[] args) {
		int max = 5;
		var actions = new System.Action[max];

		for (int i = 0; i < max; i++) {
			actions[i] = () => System.Console.WriteLine("I am action " + i);
		}

		foreach (var a in actions) a.Invoke();
	}
}
{% endhighlight %}

(Don’t worry, my actual bug was more complex than this, so please excuse me for taking some time to figure it out when it happened).

I’d naively expected that each action would print its number, 0 to 4. However, when you run this, you get “I am action 5” printed five times. That’s because at the time the Action is executed, then the variable i is actually set to 5 (which is when the condition to exit the loop is satisfied).

In order to get this to work as expected, we need to create a new variable to hold the loop index … a variable that is never mutated. We can do this inside the loop definition, because each iteration of the loop essentially gets its own variable:

{% highlight csharp %}
class Program {
	static void Main(string[] args) {
		int max = 5;
		var actions = new System.Action[max];

		for (int i = 0; i < max; i++) {
			var z = i;
			actions[i] = () => System.Console.WriteLine("I am action " + z);
		}

		foreach (var a in actions) a.Invoke();
	}
}
{% endhighlight %}

As expected, when this fixed up version runs, each `Action` correctly keeps track of its own number and outputs it to the console.

I then compared this to Java, which actually gives you some safety right out of the box. If you try to capture a mutable variable, Java code will not compile, and you get an error `java: local variables referenced from a lambda expression must be final or effectively final`. I especially like the “effectively final” bit, which means it’s not actually *required* to mark your captured variable as `final`, but as long as it’s not modified after it’s initialized, Java is ok with it. Here’s the Java code for comparison:

{% highlight java %}
public class Main {
	public static void main(String[] args) {
		int max = 5;
		var actions = new Runnable[max];

		for (int i = 0; i < max; i++) {
			int z = i;
			actions[i] = () -> System.out.println("I am action " + z);
		}

		for (Runnable a : actions) a.run();
	}
}
{% endhighlight %}

So, what if I do actually want to have all the actions in the Java version print 5, the variable at the end of the loop? Well, the correct way to capture a mutable variable is actually by a parameter to the lambda … it’s much easier to rationalize about stuff then. Like this:

{% highlight java %}
import java.util.function.IntConsumer;

public class Main {
	public static void main(String[] args) {
		int max = 5;
		var actions = new IntConsumer[max];

		int i = 0;
		for (i = 0; i < max; i++) {
			actions[i] = (x) -> System.out.println("I am action " + x);
		}

		for (var a : actions) a.accept(i);
	}
}
{% endhighlight %}

That’s a weird example, but you get the picture. Instead of using `Runnable`, I’ve used `IntConsumer` and pass in the `int` at the time we execute the lambda, rather than have it being captured. The final snippet above will have the output of my initial bugged C#, i.e. “I am action 5” repeated five times.

I’m a big fan of the way Java prevents you from shooting yourself in the foot like this.
