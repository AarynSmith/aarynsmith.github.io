---
title: "Project Euler 1 - 3"
subtitle: Solving problems 1, 2 and 3 for Project Euler
date: 2018-05-14
tags: ["python", "euler"]
draft: false
comments: true
---

In this post we will look at the first three problems on [Project Euler](projecteuler.net).
<!--more-->

I originally started solving these problems more than a decade ago to help me learn new programming languages. I originally found the site while learning (read: teaching myself) perl, then re-started for each new language I was learning. I've gone through the first few problems several times whenever I'm learning a new language or just need to brush up one one I haven't used in a while.

In the next few posts in this series, I'll be using Python, but many of the methods I use will work in whatever language you are using.

## WARNING

There are spoilers ahead. Do not read this post unless you've already solved these problems on your own.  I don't guarantee that this is the best way to solve these problems, but it is the way I solved them. I will wrap answers in spoiler tags, however (since most answers are googleable anyway) they will still be available here. Below is an example spoiler. You can click or long press to reveal the spoiler.

{{<spoiler "This is a spoiler." >}}

Now that that's been established. Lets get started.

## Problem 1

Problem 1 gives us the example of the sum of all numbers less than 10 that are divisible by 3 or 5 is 23 (3 + 5 + 6 + 9). The question we have to answer is numbers below 1000.

This is a pretty basic introduction to some of the problems we're going to encounter. I like to start by replicating their example, then modifying my code to answer the question, if possible. So, first we'll want to start with a loop counting from 0 to 10 and determining if each number is divisible by either 3 or 5 (or both). If the number is divisible then we'll add it to a running sum variable and display that value at the end.

So, we'll initialize our sum variable `s` as 0, and start a for loop over the result from the `range` function, which is not actually a function, but a sequence. We can then determine if our current number is divisible by 3 or 5 by using a modulus (`%`) operator.

```python
s = 0
for i in range(10):
    if i%3==0 or i%5==0:
        s += i
print(s)
```

Running this gives us our example result of 23. So, modifying it to fit the problem request is as simple as adjusting our range from 10 to 1000. The answer is below.

{{< spoiler 233168 >}}

## Problem 2

Problem 2 starts with a simple description of the [Fibonacci sequence](https://oeis.org/A000045) there are multiple ways to generate this sequence but for the purposes of this problem the easiest is to assign a and b to 1, use a while loop for the upper bound, and on each iteration, assign b to a and a + b to b as seen below:

```python
a, b = 1, 1
while b < 100:
    print(b)
    a, b = b, a + b
```

Python allows concurrent assignment, so we are allowed to modify a and b at the same time without having to assign a temporary variable for in the middle. Now that we are generating the sequence, we can look at the problem request. We are looking for the sum of even valued sequence terms less than 4 million. So, instead of printing each term, we'll determine if it's even, and add it to a sum variable, and adjust our limit from 100, to 4 million.

```python
s = 0
a, b = 1, 1
while b < 4000000:
    if b % 2 == 0:
        s += b
    a, b = b, a + b
print(s)
```

Running this, gives us our answer below:

{{< spoiler 4613732 >}}

## Problem 3

Problem 3 starts with the example of finding the prime factors of 13195 which are 5, 7, 13 and 29, and them asks for the factors of 600851475143. This is the first problem where time may become a factor (pun intended) in our program run.

The easy brute force way would boil down to find the largest factor of the number, figure out if it's prime, if it's not move on to the next. However, with a number this large working our way through that many divisions, and prime calculations is going to be a real time sink. We'll have to find a more efficient way.

Once we realize that the highest prime factor can only be no more than sqrt(n) that eliminates a great deal of the space we need to check. So, we'll need to A) find a factor of our number, and B) figure out if the factor is prime. We'll start with the latter part. The [Sieve of Eratosthenes](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes) is a simple ancient algorithm for finding prime numbers up to a given integer. We can then memorize those primes for later (more efficient) lookup by storing them in a set. First we'll start with an initial prime set of `[2]` and start searching up from 3. For each number we find that is divisible by any previously found primes, we'll add two to it (skipping any even numbers), for any new primes we'll add it to our prime set, and see if our target number is divisible by it. If it is, we'll divide number by it in place. Once we reach the end of our while loop, we'll be left with the largest divisor.

```python
primes = set([2])
v = 3
n = 13195
while v * v < n:
    isPrime = True
    for k in primes:
        if v % k == 0:
            isPrime = False
            v += 2
    if isPrime:
        primes.add(v)
        if n % v == 0:
            n /= v
print(int(n))
```

Our final print gives us the expected answer of 29 from the example, so if we update `n` on line 3 to 600851475143, we get the following output:

{{< spoiler 6857 >}}

## Conclusion

That wraps up the first three problems from Project Euler. If you have any questions, please feel free to leave a comment below.
