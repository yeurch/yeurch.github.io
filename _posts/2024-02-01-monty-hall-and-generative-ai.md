---
layout: post
title:  "Monty Hall and Generative AI"
---

It's a fair bet that you've heard of the [Monty Hall problem](https://en.wikipedia.org/wiki/Monty_Hall_problem).  It became famous as a question from reader Craig F. Whitaker's letter quoted in Marilyn vos Savant's "Ask Marilyn" column in Parade magazine in 1990:

> Suppose you're on a game show, and you're given the choice of three doors: Behind one door is a car; behind the others, goats. You pick a door, say No. 1, and the host, who knows what's behind the doors, opens another door, say No. 3, which has a goat. He then says to you, "Do you want to pick door No. 2?" Is it to your advantage to switch your choice?

In short, if you don't switch your choice, your chances of winning are 1/3, whereas perhaps a little counter-intuitively, if you _do_ switch, your chances increase to 2/3. This isn't a post about why that's the case ... please do go ahead and do your own thought experiment, or read about it some more in the Wikipedia article linked above.

No, what this post is about is programming. My 15-year-old daughter is doing an GCSE in Computing, and learning Python.  The Monty Hall problem came up in conversation a few days ago so I suggested to her that we programmed a simulation of the problem in Python.  We spent maybe a half-hour writing some Python that played the game 10,000 times switching, and 10,000 times not switching.  It was really satisfying for my daughter to see the numbers come out pretty close to what was expected ... around 3,300 wins if the program doesn't switch boxes, and around 6,700 wins if it switches.  Who am I even kidding, it was satisfying for me too.

Now, I've been playing around in my free time with the generative AI [ChatGPT](https://chat.openai.com) for the last week or so.  I don't have a subscription, so I'm limited to the older "inferior" ChatGPT 3.5 on my free account.  I've heard it can generate source code in various languages, but until tonight, I'd never put it to the test.  This seemed like a great opportunity to give it a try.  Here's what I asked ChatGPT:

> Write a Python program to calculate the probability of winning the Monty Hall box problem if you swap boxes, and also if you don't swap.

Quick as a flash, the AI responded with a brief outline of the problem, and a Python program to simulate it.  What took my daughter and me half an hour would have likely taken me five to ten minutes on my own.  ChatGPT gave a correct answer almost instantaneously; I ran the program it generated and it gave the expected statistics immediately.  I'll leave you now with the wise words of ChatGPT:

Here's a link to the transcript of our conversation: https://chat.openai.com/share/b19cce6f-11e6-4889-9d01-f40ccf83af61.

## ChatGPT Response

> Certainly! The Monty Hall problem is a famous probability puzzle. In the problem, a contestant is presented with three doors. Behind one door is a car (the prize), and behind the other two doors are goats. The contestant initially picks one door, and then the host (Monty Hall) opens one of the other two doors, revealing a goat. The contestant is then given the option to either stick with their original choice or switch to the other unopened door. The question is: what is the probability of winning if you stick with your initial choice versus if you switch?

> Here's a Python program to simulate and calculate the probabilities for both strategies:

```python
import random

def monty_hall_simulation(num_trials):
    stick_wins = 0
    switch_wins = 0

    for _ in range(num_trials):
        # Initialize doors with one car and two goats
        doors = ['goat', 'goat', 'car']
        
        # Contestant makes an initial choice
        contestant_choice = random.randint(0, 2)
        
        # Monty Hall opens a door with a goat
        remaining_doors = [i for i in range(3) if i != contestant_choice and doors[i] == 'goat']
        monty_opens = random.choice(remaining_doors)
        
        # Contestant chooses to either stick or switch
        stick_choice = contestant_choice
        switch_choice = [i for i in range(3) if i != contestant_choice and i != monty_opens][0]
        
        # Check if the contestant wins by sticking
        if doors[stick_choice] == 'car':
            stick_wins += 1
        
        # Check if the contestant wins by switching
        if doors[switch_choice] == 'car':
            switch_wins += 1
    
    # Calculate probabilities
    stick_probability = stick_wins / num_trials
    switch_probability = switch_wins / num_trials
    
    return stick_probability, switch_probability

if __name__ == "__main__":
    num_trials = 100000  # You can adjust the number of trials
    stick_prob, switch_prob = monty_hall_simulation(num_trials)
    
    print(f"Probability of winning by sticking: {stick_prob:.4f}")
    print(f"Probability of winning by switching: {switch_prob:.4f}")
```

> This program simulates the Monty Hall problem for a specified number of trials and calculates the probabilities of winning by sticking or switching. Keep in mind that the actual probabilities are approximately 1/3 for sticking and 2/3 for switching, demonstrating the advantage of the switching strategy.
