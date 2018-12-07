---
title: "Advent of Code 2018 Day 6"
subtitle: ""
date: 2018-12-06
tags: ["AdventOfCode", "Go"]
draft: false
comments: true
---

It's that time of year again, Christmas is around the corner and Eric Wastl has gifted us with another set of Advent of Code puzzles.
<!--more-->
Welcome to [Advent Of Code 2018](https://adventofcode.com/2018/). You can read Part 1 of this series [Here](/blog/advent-of-code-2018-day-1/) or see all the parts of this series [Here](/tags/adventofcode/).
 
## Part 1

[Day 6](https://adventofcode.com/2018/day/6) gives us a list of points an asks us to find the area of the largest interior (finite) area or zone. We'll start by reading in the points, and splitting it by line.

```go
package main

import (
	"fmt"
	"io/ioutil"
	"strings"
	"strconv"
)


func main() {
	dat, err := ioutil.ReadFile("Day6.txt")
	if err != nil {
		panic(err)
	}
	pts := strings.Split(string(dat), "\n")
    ...
}
```

Next we'll create a struct to define our Points and Zones. Our Point struct will have the X and Y coordinates. The Zone struct will consist of the area of the zone, and a boolean value to indicate if the zone continues forever outside of our grid.

```go
type Point struct {
	x, y int
}

type Zone struct {
	area int
	isInf bool
}
```

We'll use our new types to create a slice of Points, and a map from Points to Zones. We'll also track the largest X or Y coordinates to limit our for loops later on. Now, we can iterate over our points, splitting them into x and y and converting those values into integers. We'll create our Point struct, and Zone, and append the point to our Points slice. Then we'll check if the x or y is larger than our maxXY size.

```go
	Points := []Point{}
	Zones := map[Point]Zone{}
	maxXY := 0
	for _, p := range pts{
		if len(p) == 0 {
			continue
		}
		pts := strings.Split(strings.TrimSpace(p), ", ")
		x, _ := strconv.Atoi(pts[0])
		y, _ := strconv.Atoi(pts[1])
		point := Point{ x: x, y: y }
		Zones[point] = Zone{ }
		Points = append(Points, point)
		if maxXY < x { maxXY = x }
		if maxXY < y { maxXY = y }
	}
```

Next we'll write a function to determine the closest point(s) to a given x/y location using the [Manhattan Distance](https://en.wikipedia.org/wiki/Taxicab_geometry) with the following formula.

$$ Dist = | x_1 - x_2 | + | y_1 - y_2| $$ 

We'll keep track of the distances using a map of points to the distance, and track which distance is the shortest, in case the point has more than one closest point. We'll then range over the Dists map, appending any points that share the shortest distance to the Points slice for returning. 

```go
func ClosestPoint(x,y int, p []Point) (Points []Point) {
	Dists := map[Point]int{}
	ShortestDist := -1
	for _, q := range p {
		d := abs(x - q.x) + abs(y - q.y)
		Dists[q] = d
		if ShortestDist == -1 || d < ShortestDist {
			ShortestDist = d
		}
	}
	for k,v := range Dists{
		if v == ShortestDist{
			Points = append(Points, k)
		}
	}
	return Points
}
```

Now we'll check all the other points in our "grid" and find the closest point(s). If there is just one closest point, we'll add one to the area for that point's zone. We'll also set the zone as infinite if it borders the edge of our grid.

```go
	for y := 0; y < maxXY; y++ {
		for x := 0; x < maxXY; x++ {
			pts := ClosestPoint(x, y, Points)
			if len(pts) == 1 {
				z := Zones[pts[0]]
				z.area += 1
				if y == 0 || y == maxXY || x == 0 || x == maxXY {
					z.isInf = true
				}
				Zones[pts[0]] = z
			}
		}
	}
```

Once we have the size of each area, we can iterate over each of the zones, and find the largest, finite area, giving us the answer to part 1.

```go
	LargestIntZone := 0
	for _,v := range Zones {
		if !v.isInf && LargestIntZone < v.area {
			LargestIntZone = v.area
		}
	}
	fmt.Printf("P1Answer: %d\n", LargestIntZone)
```

## Part 2

For part 2, we need to find the area closest to all points, with a combined Manhattan distance of less than 10000. We'll start with a function that calculates the sum of distances to all points.

```go
func DistToAll(x,y int, p []Point) (int) {
	sum := 0
	for _, q := range p {
		d := abs(x - q.x) + abs(y - q.y)
		sum += d
	}
	return sum
}
```

Now that we can find the distances, to all points, we'll calculate that sum for each point in the grid, and add 1 to an area where that distance is less than 10000. That area is our answer to part 2.

```go
	P2Area := 0
	for y := 0; y < maxXY; y++ {
		for x := 0; x < maxXY; x++ {
			d := DistToAll(x, y, Points)
			if d < 10000 {
				P2Area += 1
			}
		}
	}
	fmt.Printf("P2Answer: %d\n", P2Area)
```

The complete code for this puzzle is available, along with the other days, on my Gitlab repository [https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_6/Day6.go](https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_6/Day6.go)

If you have any questions or comments, please feel free comment below.
