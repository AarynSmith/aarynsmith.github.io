---
title: "Advent of Code 2018 Day 2"
subtitle: ""
date: 2018-12-02
tags: ["AdventOfCode", "Go"]
draft: false
comments: true
---

It's that time of year again, Christmas is around the corner and Eric Wastl has gifted us with another set of Advent of Code puzzles.
<!--more-->
Welcome to [Advent Of Code 2018](https://adventofcode.com/2018/). You can read Part 1 of this series [Here](/blog/advent-of-code-2018-day-1/) or see all the parts of this series [Here](/tags/adventofcode/).

## Part 1

[Day 2](https://adventofcode.com/2018/day/2) starts as day 1 did with a simple task of reading a list from an input file, however instead of numbers, today's list contains a series of alphabetic locker IDs. We'll read in the file the same way we did yesterday.

```go
package main

import(
    "fmt"
    "strings"
    "io/ioutil"
)

func main(){
    dat, err := ioutil.ReadFile("Day2.txt")
    if err != nil {
        panic(err)
    }
    lines := strings.Split(string(dat), "\n")
    ...
}
```

Again, this will give us the lines from our file in a slice called lines. We'll then go on to scan through the array tallying words with 2 or 3 letters that match. To accomplish this we can scan each letter of each word and increment a map[letter]int entry to track how many times we've seen each letter in a word.

```go
    for _,i := range lines {
        letterMap := map[rune]int{}
        for _,l := range i {
            letterMap[l] += 1
        }
        ...
    }
```

Next we can look at the values in the map to determine if the word should count toward our 2 letter tally and/or our 3 letter tally.

```go
    twoCount, threeCount := 0,0
    for _,i := range lines {
        letterMap := map[rune]int{}
        for _,l := range i {
            letterMap[l] += 1
        }
        twoWord, threeWord := 0,0
        for _,v := range letterMap {
            if v == 2 {
                twoWord = 1
            }
            if v == 3 {
                threeWord = 1
            }
        }
        twoCount += twoWord
        threeCount += threeWord
    }
```

Once we've tallied all the words, we can calculate the "checksum" by multiplying our 2 letter tally by our 3 letter tally, thus completing the first part of the challenge.

```go
    fmt.Printf("Checksum: %d\n", twoCount * threeCount)
```

## Part 2

Next we need to find from our list of lockers two lockers who's IDs differ by only one letter. For this we'll write a short helper function to compare two words, as well as returning the "combined" locker ID, i.e. concatenating all of the common letters, and removing the letter that doesn't match.

```go
func compareWords(a, b string) (diff int, comb string){
    for i := 0; i < len(a); i++ {
        if (b[i] != a[i]) {
            diff++
        } else {
            comb = comb + string(a[i])
        }
    }
    return diff, comb
}
```

From here, we can set up a double for loop to iterate through the list of IDs twice, skipping IDs that match exactly and compare the ID from the first for loop, to the ID from the second for loop. If the difference is less than 2, We've found the two lockers we're looking for, and can print the combined locker ID, and exit immediately. If we don't exit, we'll get a duplicate find for when the second locker comes up in the first loop.

```go
    for _, wordOne := range lines {
        for _, wordTwo := range lines {
            if (wordOne == wordTwo || len(wordOne) != len(wordTwo)) {
                continue
            }
            diff, comb := compareWords(wordOne, wordTwo)
            if (diff < 2) {
                fmt.Printf("Found Lockers: %s\n", comb)
                return
            }
        }
    }
```

The complete code for this puzzle is available, along with the other days, on my Gitlab repository [https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_2/Day2.go](https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_2/Day2.go).

If you have any questions or comments, please feel free comment below.
