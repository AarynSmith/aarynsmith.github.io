---
title: "Advent of Code 2018 Day 10"
subtitle: ""
date: 2018-12-10
tags: ["AdventOfCode", "Go"]
draft: false
comments: true
---

It's that time of year again, Christmas is around the corner and Eric Wastl has gifted us with another set of Advent of Code puzzles.
<!--more-->
Welcome to [Advent Of Code 2018](https://adventofcode.com/2018/). You can read Part 1 of this series [Here](/blog/advent-of-code-2018-day-1/) or see all the parts of this series [Here](/tags/adventofcode/).
 
## Part 1

[Day 10](https://adventofcode.com/2018/day/10) gives us a list of star positions and velocities, and asks us to track the movements of the stars until they spell out a message. The example message resolves in 3 steps, however we're told that our actual message will take much longer to appear. We start as usual with reading our input file, spliting it by line, and creating a slice to store our Points in. Our Point type is a struct that contains a point's x and y position, as well as the x and y velocities. We'll also setup a regular expression for parsing the data for each point.

```go
type Point struct {
	x,y int
	vx, vy int
}


func main() {
	dat, err := ioutil.ReadFile("Day10.txt")
	if err != nil { panic(err) }
	lines := strings.Split(string(dat), "\n")

	Points := []Point{}

	PointRegexp := regexp.MustCompile(
		`position=<\s?([-0-9]+),\s\s?([-0-9]+)>\s` +
		`velocity=<\s?([-0-9]+),\s\s?([-0-9]+)>`,
	)
    ...
}
```

Next we'll iterate over the lines, and find our regexp matches for each line. If we find a match we'll convert each element to an integer and append our new point to our slice.

```go
	for i, l := range lines {
		if len(l) == 0 { continue }
		m := PointRegexp.FindStringSubmatch(l)
		if m == nil { panic(fmt.Errorf("Line %d: No match found",i)) }
		x, err := strconv.Atoi(m[1])
		if err != nil { panic(err) }
		y, err := strconv.Atoi(m[2])
		if err != nil { panic(err) }
		vx, err := strconv.Atoi(m[3])
		if err != nil { panic(err) }
		vy, err := strconv.Atoi(m[4])
		if err != nil { panic(err) }
		Points = append(Points, Point{
			x: x,
			y: y,
			vx: vx,
			vy: vy,
		})
	}

```

In order to determine when a message is fully formed, we need to determine what the size of the current message is. As the message is forming, the overall area decreases, since the stars are moving closer together. We should be able to tell if the message is complete, if the stars start diverging, and the area increases. So, we'll write an absolute value function, which returns the postive of a number it takes in, and a function that will calculate the total area for a message, by finding the minimum and maximum x and y values, and adding them together to get a x and y range. 

```go
func abs(i int) int {
	if i < 0 {
		return i * -1
	}
	return i
}

func CalcSize(Points []Point) (gMaxX, gMaxY int) {
	minx, maxx, miny, maxy := 1<<63 - 1, -1 << 63, 1<<63 -1, -1 << 63
	for _, p := range Points {
		if p.x < minx { minx = p.x }
		if p.y < miny { miny = p.y }
		if p.x > maxx { maxx = p.x }
		if p.y > maxy { maxy = p.y }
	}
	gMaxX = abs(maxx) + abs(minx)
	gMaxY = abs(maxy) + abs(miny)
	return
}
```

Next we'll need a function to progress our stars. We'll iterate over the slice and add the x velocity to the x position, and y velocity to the y position.

```go
func MovePoints(Points []Point){
	for i := 0; i < len(Points); i++{
		Points[i].x += Points[i].vx
		Points[i].y += Points[i].vy
	}
}
```

Now that we can calculate the size, and move the points, we can setup a loop that will calculate the move the points, and calculate the size, while keeping track of the previous positions. Once we reach a point where the stars start diverging, we'll break the loop, and our answer will be held in the prevS variable. However we'll want function to render the message in a readible form.

```go
	x,y := CalcSize(Points)
	s := x * y
	prevS, prevPoints := s-1, Points
	for {
		prevS, prevPoints = s, Points
		MovePoints(Points)
		x, y = CalcSize(Points)
		s = x * y
		if s > prevS {
			break
		}
	}
```

Our RenderPoints function finds the size of our message, then creates a two-dimensional slice of strings to store the position of the stars. Once the stars have ben plotted, the function iterates through the Grid, and uses Sprintf to append the current position to a string. This will successfully render the message, however at this point our stars have traveled quite a distance so we'll need to trim some whitespace from the end of the grid, we'll iterate over the lines of the msg removing any blank lines and trimming trailing spaces from the line.

```go
func RenderPoints(Points []Point) (msg string) {
	minx, maxx, miny, maxy := 1<<63 - 1, -1 << 63, 1<<63 -1, -1 << 63
	for _, p := range Points {
		if p.x < minx { minx = p.x }
		if p.y < miny { miny = p.y }
		if p.x > maxx { maxx = p.x }
		if p.y > maxy { maxy = p.y }
	}
	gMaxX := abs(maxx) + abs(minx)
	gMaxY := abs(maxy) + abs(miny)

	Grid := make([][]string, gMaxX+1)
	for i := range Grid {
		Grid[i] = make([]string, gMaxY+1)
	}

	for _, p := range Points{
		Grid[p.x - abs(minx)][p.y - abs(miny)] = "*"
	}

	for y := 0; y <= gMaxY; y++{
		for x := 0; x <= gMaxX; x++{
			msg += fmt.Sprintf("%1s", Grid[x][y])
		}
		msg += fmt.Sprintf("\n")
	}
	msg, trim := "", msg
	for _, l := range strings.Split(trim, "\n") {
		if strings.Contains(l, "*") {
			msg += strings.TrimSpace(l)+"\n"
		}
	}
	return
}
```

Now that we're able to render the message, we simply output it for the answer to part 1.

```go
	Msg := RenderPoints(prevPoints)
	fmt.Printf("P1Answer:\n%s\n",Msg)
```

## Part 2

Part 2 asks us exactly how long the message would have taken to resolve, had we not sped up the process by precalculating the paths. This is as simple as adding a counter to our main movement loop and displaying it at the end.

```go
    i := 0
	for {
		prevS, prevPoints = s, Points
		MovePoints(Points)
		i += 1
		x, y = CalcSize(Points)
		s = x * y
		if s > prevS {
			break
		}
	}

	Msg := RenderPoints(prevPoints)
	fmt.Printf("P1Answer:\n%s\n",Msg)
	fmt.Printf("P2Answer: %d\n", i)
```

The complete code for this puzzle is available, along with the other days, on my Gitlab repository [https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_10/Day10.go](https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_10/Day10.go)

If you have any questions or comments, please feel free comment below.
