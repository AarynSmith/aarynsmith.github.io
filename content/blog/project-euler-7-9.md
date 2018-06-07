---
title: "Project Euler 7 - 9"
subtitle: Solving problems 7, 8, and 9 for Project Euler
date: 2018-06-07
tags: ["python", "euler"]
draft: false
comments: true
---

In this post we will look at problems 7, 8, and 9 on [Project Euler](projecteuler.net).
<!--more-->

The same warning applies to this post as the [first post in this series](/blog/project-euler-1-3/#warning)

## Problem 7

Problem 7 gives us an example of the first 6 primes, and asks us to find the 10001st prime. We can easily find a way to count primes using a while loop, and verify that an incremented number is by writing a simple primailty test.

In this example our `isPrime()` function first eleminates 2 and then any even numbers, then use the any function to iterate over a range of 3 to the square root of our number plus 1, incremened by 2s, to get all odd numbers between 3 and the square root of a + 1. `a % x == 0` checks to see if a is divisable by the numbers in the range, and if any are, returns `True`, which is then negated by the not clause.

```python
def isPrime(a):
    if a == 2:
        return True
    if a < 2 or a % 2 == 0:
        return False
    return not any(a % x == 0 for x in range(3, int(a**0.5) + 1, 2))


PrimeCount = 0
i = 0
while PrimeCount < 6:
    i += 1
    if isPrime(i):
        PrimeCount += 1

print(i)
```

This gives us our example of 13, and if we modify the while loop to be 10001 instead of 6 we get the following solution:

{{< spoiler 104743 >}}

## Problem 8

Problem 9 gives us a 1000 digit number and tells us the greatest product of 4 adjacent digits is 5832 (9 x 9 x 8 x 9). Our task is to find the greatest product of 13 adjacent cells.

While this is technically a 1000 digit number, that does not necessarily mean we have to treat it as a number. In this case it would be easier to treat it as a string, and then iterate over our characters (digits) to find our adjacent numbers.

First we'll set up a short function that will multiply the digits in a string together and return the product. Next we'll have to create a variable with our 1000 digit number in it with a series of concatenations via the `+=` operator. Next we'll setup a for loop that will give us an index up to the end of the string, excluding the length of adjacent number of digits we are looking for (to prevent reading past the end of our string). We can then feed a slice of our string into our `AdjProduct()` function and track the maximum result.

```python
def AdjProduct(n):
    prod = 1
    for i in n:
        prod *= int(i)

    return prod


BigNum = "73167176531330624919225119674426574742355349194934"
BigNum += "96983520312774506326239578318016984801869478851843"
BigNum += "85861560789112949495459501737958331952853208805511"
BigNum += "12540698747158523863050715693290963295227443043557"
BigNum += "66896648950445244523161731856403098711121722383113"
BigNum += "62229893423380308135336276614282806444486645238749"
BigNum += "30358907296290491560440772390713810515859307960866"
BigNum += "70172427121883998797908792274921901699720888093776"
BigNum += "65727333001053367881220235421809751254540594752243"
BigNum += "52584907711670556013604839586446706324415722155397"
BigNum += "53697817977846174064955149290862569321978468622482"
BigNum += "83972241375657056057490261407972968652414535100474"
BigNum += "82166370484403199890008895243450658541227588666881"
BigNum += "16427171479924442928230863465674813919123162824586"
BigNum += "17866458359124566529476545682848912883142607690042"
BigNum += "24219022671055626321111109370544217506941658960408"
BigNum += "07198403850962455444362981230987879927244284909188"
BigNum += "84580156166097919133875499200524063689912560717606"
BigNum += "05886116467109405077541002256983155200055935729725"
BigNum += "71636269561882670428252483600823257530420752963450"

adjNum = 4
max = 0
for i in range(len(BigNum) - adjNum + 1):
    m = AdjProduct(BigNum[i:i+adjNum])
    if m > max:
        max = m

print(max)
```

Running this gives us the example output of 5832, and if we update the `adjNum` to 13 we get the following result:

{{< spoiler 23514624000 >}}

## Problem 9

Problem 9 gives us an example of a pythagorean triplet below:

$$ a^2 + b^2 = c^2$$
$$\text{where:  } a < b < c $$

We are then asked for the product of a * b * c where a + b + c = 1000.

We can start with a couple of nested for loops, one (`b`) going to our limit, for the examle we'll use a sum of 15 to include the given numbers of 12, and a second for loop (`a`) that limits at the previous to prevent a from being greater than b. We can then calculate `c` using `(a**2 + b**2)**.5`.

```python
limit = 12
for b in range(1, limit):
    for a in range(1, b):
        c = (a**2+b**2)**.5
        if int(c) == c and a + b + c == limit:
            print(int(a*b*c))
```

Running this gives us our example 60 (3 * 4 * 5), changing our limit to 1000 gives us the answer below:

{{< spoiler 31875000 >}}

## Conclusion

That wraps up the third set of three problems from Project Euler. If you have any questions, please feel free to leave a comment below.
