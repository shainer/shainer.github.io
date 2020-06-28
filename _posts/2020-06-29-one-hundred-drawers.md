---
layout: post
title:  "100 drawers"
date:   2020-06-29 08:30:00 +0100 
categories: math 
---

And as I promised in the previous post, I did some research, lots of furious scribbling on paper, and now I am ready to present a new interesting math problem. This is, again, taken from the book _So you think you have problems?_ by Alex Bellos (and from the same section).

Sadly, this time I cannot claim of having solved the problem myself. The solution required either familiarity with a branch of math I've never really explored much before, or a non-negligible jump in intuition. I had neither and resorted to reading the solution at the end of the book. Still, as Alex Bellos says himself, the solution was surprising enough
to pique my curiosity. Bellos explains the solution but purposedly leaves out the mathematical proof, as too complex
for the intended audience of the book.

Finally, I am not attaching any code to the solution. Perhaps I will implement some handy computations in the following days if I feel like it, but that's not necessary to understand all this anyway, so don't hold your breath :)

# The problem: "100 drawers"

100 prisoners are jailed inside a prison. The wardens, ever the joking types, decide to give them a chance at freedom: they place 100 drawers inside an otherwise empty room, then write each of the 100 prisoners' name on a piece of paper. Finally, they place each piece of paper in a drawer, at random.

Next, the prisoners are allowed to enter the room one by one; they can open any 50 drawers they wish, then leave. If all 100 of them have found their name among the 50 drawers they opened, they are all freed. If even one of them did not find his own name, they will all be executed.

The prisoners are allowed to discuss and agree on a strategy in advance, but once they start entering the room, any further communication (including indirect ones via the objects in the room) is forbidden. This means they cannot leave any clues to each other as to which drawers they decided to open or what they found inside.

If all prisoners open 50 drawers at random, each one of them will have a 0.5 chance of finding his own name. The chance of all of them finding their own names are therefore `0.5 * 0.5 ... 100 times`, which means `0.5 ^ 100`, a very very small number. However, there is a strategy that raises their possibility of winning this game to approximately 30%. Can you find such a strategy?

Stop here if you want to think about this for yourself. From now on, we'll discuss solutions. Spoilers alert.

# The solution

The first thing to note is that the problem works exactly the same way if we reduce the numbers to a more manageable level: let's have 10 prisoners, 10 drawers, with each prisoner opening 5 drawers. The probability for the "baseline" random strategy (`0.5 ^ 10`) is still very tiny, so being able to raise it to about 30% is equally desirable.

Let's pretend our prisoners are named with numbers from 1 to 10. Now, when the warders are placing the names (pardon, numbers) in the drawers, they are creating a **permutation** of the set of numbers from 1 to 10.

There's a whole branch of math studying permutations; in particular, we are interested to permutation cycles and their properties.

## Permutation cycles

Let's take the following permutation of the numbers from 1 to 10.

|                      |                               |
| -------------------- | ----------------------------- |
| **Original numbers** | 1  2  3  4  5  6  7  8  9  10 |
| **Permutation**      | 4  8  5  6  9  1  10 2  7  3  |

Examining it from number 1, we see that 1 goes to 4, 4 goes to 6, 6 goes to 1. This means that {4 -> 6 -> 1 -> 4 ...} is
a cycle of this permutation. The remaining numbers are also arranged into cycles, namely {2 -> 8 -> 2 ...} and {3 -> 4 -> 9 -> 7 -> 10 -> 3}. The number of elements within a single cycle is called the _length_ or _size_ of the cycle.

This would be much nicer to see with a drawing that shows the number within one cycle arranged in a circle, but we do what we can :)

Note that if a number's position is not changed by the permutation, it technically forms a cycle of size 1; such
cases are called _fixed points_.

All permutations can be expressed as a set of disjoint cycles, including fixed points. At one end we have the identity permutation (= the one where you don't change the position of any element), with all the cycles being fixed points, while at the other we have the so called _cyclic permutations_, with all the elements arranged in one big cycle of size n.

There's a whole branch of math theory studying permutations and cycles (who knew?). We'll derive a few properties
later on, since they will be useful for our purposes. For now, let's return to our problem.

## The strategy

Let's look again at our example permutation. Imagine the first row represents the numbered drawers, and the second row 
tells us which prisoner's number is inside each drawer.

|                      |                               |
| -------------------- | ----------------------------- |
| **Original numbers** | 1  2  3  4  5  6  7  8  9  10 |
| **Permutation**      | 4  8  5  6  9  1  10 2  7  3  |

This means drawer 1 has the name of prisoner 4, drawer 2 has the name of prisoner 8, etc.etc.

Prisoner number 1 enters the room. The strategy goes like this: first, he opens drawer 1. He does not find his name, instead he finds prisoner 4's name. So his next move is to open drawer number 4. Following the same logic, he then opens drawer 6, and there he finds his name.

You can see that since the permutation contains the cycle {4 -> 6 -> 1 -> 4 ...}, prisoners number 1, 4 and 6 can follow this strategy and always find their name in exactly 3 steps. This is because they start by opening the drawer directly connected to the one containing their name (i.e. on the "receiving end" of the arrow). Prisoner 1 will open drawer 1, which is on the receiving end of the arrow coming from drawer 6 (the one containing his name). Then, he will keep following the arrow to drawer 4 and finally to drawer 6. As the arrows are uni-directional, he cannot move in the other direction and find the correct drawer without going through the whole cycle.

With the same logic, prisoners 2 and 8 will find their names in 2 steps, and all the others in 5 steps.

Since each prisoner is allowed to open at most 5 drawers, it follows that this strategy is guaranteed to work if the permutation implemented by the wardens happens to only contain cycles of 5 or less elements, as in our example above, and is guaranteed not to work if the permutation has one cycle with more than 5 elements.

To accept this strategy as valid, we need to prove that the percentage of permutations of 10 elements with only cycles of size 5 or less is about 35% of the total number of possible permutations. This is where the book stopped, and where things get a bit more technical. Prepare to brush off some combinatorics!

## The proof

Some terminology: let's call n the total number of elements (10 in our problem) and k the length of the cycle under consideration.

If a permutation has a cycle with length k > n/2, it must have only one of them, for obvious reasons. My intuition told me that counting the permutations that have one cycle bigger than n/2 would therefore be easier, since I don't need to care about the other cycles such permutations contain.

**Claim**: for every k > n/2, the number of permutations with a cycle of length k is [formula 1]({{ site.url }}/images/one_hundred_drawers/formula1.png)

Let's focus on a single value of k (doesn't matter which one). Combinatorics tells me how many ways I have of selecting k elements from a larger set of n elements, when the order of the elements matter:

[formula 2]({{ site.url }}/images/one_hundred_drawers/formula2.png)

Now I have selected k elements. There are k! permutations of such elements; how many of them can be considered one big cycle? In other words, how many permutations of k elements are a cyclic permutation? Let's say that it is our convention to always look at cycles as starting from element a, despite its actual position in the list; this works because I can always "rotate" my cycle to place a wherever I want. Then, I can permute all the remaining elements in any way and still get a cycle. This means that the number of cyclic permutations is (k-1)!

Combining the two things together, I select k elements and then arrange them in a cycle in any way possible. This gives me this many combinations:

[formula 3]({{ site.url }}/images/one_hundred_drawers/formula3.png)

But wait, I have other (n-k) elements in my original set. Once I've established that a given permutation has a cycle of length k, I don't really care how the remaining (n-k) elements are arranged, i.e. any possible arrangement still gives me a "valid" permutation. So the above expression becomes:

[formula 4]({{ site.url }}/images/one_hundred_drawers/formula4.png)

which gives me the stated result after simplifications.

Now if I were to be fancy, I would go ahead and look at how the formula works for k < n/2, i.e. when you can have multiple cycles of the length I am looking for. That's for another day I guess :)

Ok, claim proved. Now, I have a formula for the number of permutations with a cycle of length k; as mentioned, I need to sum this over all values of k from n/2+1 up to n.

[formula 5]({{ site.url }}/images/one_hundred_drawers/formula5.png)

1/k for a given value of k is called **harmonic number**, or H(k). The sum above comes up to H(n) - H(n/2 + 1). I am not sure if there's an efficient formula for a generic H(k), but worst case you can always compute the whole sum.

This final value also featured on on-line encyclopedia of integer sequences: https://oeis.org/A201546. Harmonic numbers are featured too, but of course with two different sequences, one for the denominators and one of the numerators (or they wouldn't be integer sequences at all...).

Having computed the number of permutations of n elements with a cycle of length greater than n/2, I need the opposite: I need all the permutations that DON'T have a cycle of length greater than n/2, which means subtracting this formula I've just found to the total number of permutations of n elements (n!).

[formula 6]({{ site.url }}/images/one_hundred_drawers/formula6.png)

And finally you can be fancy and compute percentages and whatnot. I'll leave that to you :) Plugging the numbers for the problem with 10 drawers gives about 35%, as promised. Plugging the numbers for n=12 already yields something noticeably smaller, about 34.6%. I can assume this gets even smaller as n increases, and settles to around 30% by the time n gets to 100. Why is that? I don't know :) maybe that could be homework for next time...