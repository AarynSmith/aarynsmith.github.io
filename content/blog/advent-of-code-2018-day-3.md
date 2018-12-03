---
title: "Advent of Code 2018 Day 3"
subtitle: ""
date: 2018-12-03
tags: ["AdventOfCode", "Go"]
draft: false 
comments: true
---

It's that time of year again, Christmas is around the corner and Eric Wastl has gifted us with another set of Advent of Code puzzles.
<!--more-->
Welcome to [Advent Of Code 2018](https://adventofcode.com/2018/). You can read Part 1 of this series [Here](/blog/advent-of-code-2018-day-1/) or see all the parts of this series [Here](/tags/adventofcode/).
 
## Part 1

[Day 3](https://adventofcode.com/2018/day/3) starts again with reading some data from a text file. Today we have a list of claims for a square piece of cloth 1000 inches on each side. Each claim has an ID number, a starting x and y position, and a size. First we'll read in the file.

```go
package main

import(
    "fmt"
    "strings"
    "io/ioutil"
    "regexp"
    "strconv"
)

func main(){
    dat, err := ioutil.ReadFile("Day3.txt")
	if err != nil {
	    panic(err)
	}
	claims := strings.Split(string(dat), "\n")
    ...
}
```

From here we'll initialize a 1000x1000 2D array to process our claims into.

```go
	Cloth := make([][]int, 1000)
	for i := range Cloth {
		Cloth[i] = make([]int, len(Cloth))
	}
```

This will result in a 1000x1000 array filled with 0's. Since the IDs are all positive integer values, we can store the IDs in each square inch that the claim has made. For overlapping claims, we'll change that square inch to -1 and not reassign it in the future.

The next step we need to do is parse each line of the input file and proccess each claim onto our cloth. To do this we'll write a function to parse the claim, and then use that data to populate the cloth.

Here we define what a claim looks like using a struct, ID number, x and y position, and length and width. Next we deine our parseClaim function, which will take the string from the file, and return a claim struct.

We'll use regexp to find the parts of the string we're looking for, run each part through strconv's Atoi function to get integers, and populate the claim struct with the data.

```go
type claim struct {
	ClaimNo int
	x, y int
	dx, dy int
}

func parseClaim (c string) *claim {
	claimRegexp := regexp.MustCompile(`#(\d+)\s@\s(\d+),(\d+):\s(\d+)x(\d+)`)
	m := claimRegexp.FindStringSubmatch(c)
	if m == nil {
		return nil
	}
	cn, _ := strconv.Atoi(m[1])
	x, _ := strconv.Atoi(m[2])
	y, _ := strconv.Atoi(m[3])
	dx, _ := strconv.Atoi(m[4])
	dy, _ := strconv.Atoi(m[5])

	claim := claim{
		ClaimNo: cn,
		x: x,
		y: y,
		dx: dx,
		dy: dy,
	}
	return &claim
}
```

With this we are able to create a couple nested for loops to mark off the claims on the cloth, if the claim we are trying to make is not 0, we mark it -1 since, indicating an overlap.

```go
	for _, claim := range claims{
		c := parseClaim(claim)
		if c == nil {
			continue
		}
		for x := c.x; x < c.x + c.dx; x++{
			for y := c.y; y < c.y + c.dy; y++{
				if Cloth[x][y] == 0 {
					Cloth[x][y] = c.ClaimNo
				} else {
					Cloth[x][y] = -1
				}
			}
		}
	}
```

Next we'll iterate through both dimensions of the cloth, and count the number of overlaps (-1's) we find, giving us the answer to Part 1.

```go
	c := 0
	for i := range a {
		for j := range a[i]{
			if a[j][i] == -1 {
				c += 1
			}
		}
	}
	fmt.Printf("%d overlapping inches\n", c)
```

## Part 2

Next we need to find the one claim that does not overlap with any other claim. To do this, we can re-iterate over our claims, and verify that each square inch of the claim contains it's ID, and not -1, indicating an overlap.

We can use a boolean variable (Clean), and set it to false to indicate that the claim no longer has control over that square inch. We'll also use a label (Loop) in order to break out of both for loops on finding an unclean square inch. If we make it through the whole claim without setting clean to false, we'll have found the ID number of the unmolested claim.
```go
    for _, claim := range claims {
		c := parseClaim(claim)
		if c == nil {
			continue
		}
		Clean := true
    Loop:
		for x := c.x; x < c.x + c.dx; x++{
			for y := c.y; y < c.y + c.dy; y++{
				if Cloth[x][y] != c.ClaimNo {
					Clean = false
                    break Loop
				}
			}
		}
		if Clean {
			fmt.Printf("Claim %d is clean\n", c.ClaimNo)
		}
```

The complete code for this puzzle is available, along with the other days, on my Gitlab repository [https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_3/Day3.go](https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_3/Day3.go). 

If you have any questions or comments, please feel free comment below.

