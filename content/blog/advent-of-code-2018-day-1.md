---
title: "Advent of Code 2018 Day 1"
subtitle: ""
date: 2018-12-01
tags: ["AdventOfCode", "Go" ]
draft: false
comments: true
---

It's that time of year again, Christmas is around the corner and Eric Wastl has gifted us with another set of Advent of Code puzzles.
<!--more-->
Welcome to [Advent Of Code 2018](https://adventofcode.com/2018/). You can see all the parts of this series [Here](/tags/adventofcode/).

## Part 1

[Day 1](https://adventofcode.com/2018/day/1) kicks off with a simple task of reading a list of positive and negative numbers from an input file.

```go
package main

import(
    "strconv"
    "io/ioutil"
    "strings"
    "fmt"
)

func main() {
    dat, err := ioutil.ReadFile("Day1.txt")
    if err != nil {
        panic(err)
    }
    lines := strings.Split(string(dat), "\n")
    ...
}
```

After we have our slice of lines, we can iterate through the slice, skipping any lines that have no text. With each line, we'll convert the string to an int, and add it to our current frequency, starting at 0.

```go
CurrentFreq := 0
for idx :=0; idx < len(lines); idx +=1 {
    if len(lines[idx]) == 0 {
        continue
    }

    FreqChange, err := strconv.Atoi(lines[idx])
    if err != nil {
        panic(err)
    }

    NewFreq := CurrentFreq + FreqChange

    fmt.Printf("Current frequency %d, change of %d; resulting frequency %d.\n",
        CurrentFreq, FreqChange, NewFreq,
    )

    CurrentFreq = NewFreq
}

fmt.Printf("Final Frequency Found: %d.", CurrentFreq)
```

This will give us the answer to part 1 of the puzzle.

## Part 2

The puzzle then goes on to explain that we need to loop over the list, potentially several times, to find the first frequency that we land on twice. In order to accomplish this, we'll need to modify our for loop. Instead of stopping at the last item in the slice, we'll need to loop through the list over and over, resetting our index to 0 when it is the length of lines.

```go
CurrentFreq := 0
idx := 0
for {
    if len(lines[idx]) == 0 {
        idx = (idx + 1) % len(lines)
        continue
    }

    FreqChange, err := strconv.Atoi(lines[idx])
    if err != nil {
        panic(err)
    }

    NewFreq := CurrentFreq + FreqChange

    fmt.Printf("Current frequency %d, change of %d; resulting frequency %d.\n",
        CurrentFreq, FreqChange, NewFreq,
    )

    CurrentFreq = NewFreq
    idx = (idx + 1) % len(lines)
}

fmt.Printf("Final Frequency Found: %d.", CurrentFreq)
```

Now we will continually loop through the list of frequency changes, but we'll never reach the end of the loop, since there is no break condition. What we'll need to do next is track the frequencies we've already visited, and when we see one for a second time, we'll end the loop.

```go
    CurrentFreq := 0
    FreqMap := map[int]int{};
    FreqMap[CurrentFreq]++
    idx := 0;
    for {
        if len(lines[idx]) == 0 {
            idx = (idx + 1) % len(lines)
            continue
        }

        FreqChange, err := strconv.Atoi(lines[idx])
        if err != nil {
            panic(err)
        }

        NewFreq := CurrentFreq + FreqChange

        CurrentFreq = NewFreq
        FreqMap[CurrentFreq]++
        if (FreqMap[CurrentFreq] == 2){
            break
        }


        idx = (idx + 1) % len(lines)
    }

    fmt.Printf("Final Frequency Found: %d.", CurrentFreq)
```

This will finally result in the final frequency we are looking for. The complete code for this puzzle is available, along with the other days, on my Gitlab repository [https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_1/Day1.go](https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_1/Day1.go).

If you have any questions or comments, please feel free comment below.
