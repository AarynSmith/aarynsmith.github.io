---
title: "Project Euler 4 - 6"
subtitle: Solving problems 4, 5, and 6 for Project Euler
date: 2018-05-17
tags: ["python", "euler"]
draft: false
comments: true
---

In this post we will look at problems 4, 5, and 6 on [Project Euler](projecteuler.net).
<!--more-->

The same warning applies to this post as the [first post in this series](/blog/project-euler-1-3/#warning)

## Problem 4

This problem gives us the example of the largest palindromic number made from the product of two two digit numbers, 99 x 91 = 9009, where a palindromic number is the same read forwards and backwards. The problem then asked is what the largest palindromic number made from the product of two three digit numbers is.

This problem is fairly straight forward first we need to loop through two digit numbers, which can be easily accomplished with the `range(start,end)` function and the reversing can be easily accomplished by converting a number to a string using the `str()` function, and then reversing the string using [extended slice syntax](https://docs.python.org/2/whatsnew/2.3.html#extended-slices), `'string'[::-1]`.

```python
m = 0
for x in range(10,100):
    for y in range(10,x):
        if x*y > m and str(x*y) == str(x*y)[::-1]:
            m = x*y
print(m)
```

This gives us our example of 9009. If we modify the range lower limits from 10 to 100, and 100 to 1000, so we have a set of three digit numbers, we get the answer for the problem below.

{{< spoiler 906609 >}}

## Problem 5

This problem gives us an example of the number 2520 being the smallest number that is evenly divisible by each of the numbers between 1 and 10, and asks us for the smallest number that can be evenly divisible by each of the numbers between 1 and 20.

The example is easily solved via a brute force method as seen below, however if we increase the range from 11 to 21 to cover the problem question, the runtime increases to well over 2 minutes. We'll have to be a little more clever.

```python
def f(n):
    for x in range(1,11):
        if n % x != 0
            return False
    return True

i = 1
while not f(i):
    i += 1
print(i)
```

Investigating the problem closer we can find that we can actually find the solution via a least common multiples (LCM) of our divisors. We can write a couple basic functions to calculate first the greatest common denominator, and then the LCM. Then we can find the LCM of 1 and 2, and then the LCM of that number and each of the other divisors.

```python
def gcd(a,b):
    while b > 0:
        a, b = b, a % b
    return a

def lcm(a,b):
    return int(a*b/gcd(a,b))

num = lcm(1,2)
for i in range(2,11):
    num = lcm(num,i)
print(num)
```

This gives us our example answer of 2520, and if we update the range to 21, the new script runs in less than a second and gives our answer below.

{{< spoiler 232792560 >}}

## Problem 6

This problem's example gives us the sum of the squares of the first ten natural numbers and the square of the sum of the first ten natural numbers, as seen below. We are then asked for the difference between the sum of the squares of the first 100 natural numbers and the square of the sum of the first 100 natural numbers.

$$
1^2+2^2+ \mathellipsis + 10^2 = 385
$$
$$
(1+2+ \mathellipsis + 10 ) ^2 = 55^2 = 3025
$$

In some languages such as C++ we'd have to define our variables as at least `unsigned int`s to get the precision required for the size of numbers we are dealing with her, however since python's numeric data types have unlimited precision, this problem is fairly trivial. For the example we can iterate over the first 10 natural numbers using the `range()` function and track a sum of the numbers, and a sum of squares. Then we can take the difference of the sum of squares and the sum variable squared.

```python
squares = 0
sum = 0
for i in range(11):
    squares += i ** 2
    sum += i
print(sum ** 2, squares, sum ** 2 - squares)
```

This yields the expected difference of 2640. Updating our range from 11 to 101, gives us our answer below.

{{< spoiler 25164150 >}}

## Conclusion

That wraps up the second set of three problems from Project Euler. If you have any questions, please feel free to leave a comment below.
