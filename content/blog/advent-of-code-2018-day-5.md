---
title: "Advent of Code 2018 Day 5"
subtitle: ""
date: 2018-12-05
tags: ["AdventOfCode", "Go"]
draft: false
comments: true
---

It's that time of year again, Christmas is around the corner and Eric Wastl has gifted us with another set of Advent of Code puzzles.
<!--more-->
Welcome to [Advent Of Code 2018](https://adventofcode.com/2018/). You can read Part 1 of this series [Here](/blog/advent-of-code-2018-day-1/) or see all the parts of this series [Here](/tags/adventofcode/).

## Part 1

[Day 5](https://adventofcode.com/2018/day/4) gives us a long polymer sequence an asks us to reduce it by finding pairs of upper- and lowercase letters. We start by reading in the entire sequence, trimming any extra characters off the end.

```go
package main

import (
    "fmt"
    "io/ioutil"
    "strings"
)

func main() {
    dat, err := ioutil.ReadFile("Day5.txt")
    if err != nil {
        panic(err)
    }
    Seq := strings.TrimSpace(string(dat))
    ...
}
```

Next we'll write a quick function to find all the components our polymer is made up of, since we're only concerned an upper/lowercase pair, we'll just record the uppercase versions. We can create a map using the uppercase components as a key to ensure we get a complete list of unique components, then iterate over the map and append each component to a slice.

```go
func Components(s string) []string {
    r := map[string]bool{}
    for _, c := range s {
        r[strings.ToUpper(string(c))] = true
    }
    keys := make([]string, 0, len(r))
    for k, _ := range r {
        keys = append(keys, k)
    }
    return keys
}
```

Next we'll want to write a function to reduce our polymer, by removing adjacent upper/lower pairs. A quick and easy way to accomplish this is to use the strings.Replace method. We can feed in an old string of component + strings.ToLower(component), and a new string of "", to effectively delete any adjacent pairs. Of course we'll also need to do the inverse as well to cover any lower/upper pairs. If we range over our list of components, we can eliminate any pairs, but this could leave us with un-reduced pairs after deletion. We can compare the input length to the reduced length. If it has been reduced, we try again, making our function recursive. If it hasn't been reduced, we return our final length up the chain.

```go
func Components(s string) []string {
    r := map[string]bool{}
    for _, c := range s {
        r[strings.ToUpper(string(c))] = true
    }
    keys := make([]string, 0, len(r))
    for k, _ := range r {
        keys = append(keys, k)
    }
    return keys
}
```

```go
func Reduce(s string, comp []string) string {
    var origLen = len(s)
    for _, c := range comp {
        s = strings.Replace(s, c + strings.ToLower(c), "", -1)
        s = strings.Replace(s, strings.ToLower(c) + c, "", -1)
    }
    if origLen > len(s) {
        s = Reduce(s, comp)
    }
    return s
}
```

Running this function on our list will give us our fully reduced sequence, as well as the result for part 1.

```go
    Seq = Reduce(Seq, Components(OrigSeq))
    fmt.Printf("P1Answer: %d\n", len(Seq))
```

## Part 2

Part 2 asks us to find which component we can remove completely to reduce our polymer the most. By starting with our list of components, and iteratively removing a component's upper and lower versions from the original sequence using the strings.Replace method. Then reducing it, we can track and find the shortest possible polymer sequence. Resulting in our answer to part 2.

```go
    Comp := Components(OrigSeq)
    P2Count := len(OrigSeq)
    for _, c := range Comp {
        newSeq := strings.Replace(OrigSeq, c, "", -1)
        newSeq = strings.Replace(newSeq, strings.ToLower(c), "", -1)
        newSeq = Reduce(newSeq, Comp)

        if P2Count > len(newSeq) {
            P2Count = len(newSeq)
        }
    }
    fmt.Printf("P2Answer: %d", P2Count)
```

The complete code for this puzzle is available, along with the other days, on my Gitlab repository [https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_5/Day5.go](https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_5/Day5.go)

If you have any questions or comments, please feel free comment below.
