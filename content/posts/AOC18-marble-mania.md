---
title: "AOC18: Marble Mania"
subtitle: "Advent of Code 2018 Day 9"
date: 2018-12-09
draft: false

tags: ["AdventOfCode", "Go"]
categories: ["Blog"]

hiddenFromHomePage: false

featuredImage: ""
featuredImagePreview: ""

comments:
  enable: true
toc:
  enable: true
  auto: false
math:
  enable: true
---

It's that time of year again, Christmas is around the corner and Eric Wastl has gifted us with another set of Advent of Code puzzles.
<!--more-->
Welcome to [Advent Of Code 2018](https://adventofcode.com/2018/). You can read Part 1 of this series [Here](/blog/advent-of-code-2018-day-1/) or see all the parts of this series [Here](/tags/adventofcode/).

## Part 1

[Day 9](https://adventofcode.com/2018/day/9) teaches us how to setup and play a popular marble game among the elves. The idea behind the game is each player places a numbered marble 1 place clockwise from the previously placed marble, unless the the marble's number is a multiple of 23, in which case the marble is kept, and the marble 7 places counter clockwise is removed and kept. The next marble would be placed 1 place from the one that was removed. Play continues until all the marbles are used.

We begin today's puzzle in a similar fashion to previous day's puzzles. We'll read our input file, use a regexp to find the number of players and marbles, and convert them from strings into integers.

```go
func main(){
    dat, err := ioutil.ReadFile("Day9.txt")
    if err != nil { panic(err) }

    inpRegexp := regexp.MustCompile(`(\d+) players; last marble is worth (\d+) points`)
    m := inpRegexp.FindStringSubmatch(string(dat))
    if m == nil { panic("Couldn't Parse Input") }

    players, err := strconv.Atoi(m[1])
    if err != nil { panic(err) }
    marbles, err := strconv.Atoi(m[2])
    if err != nil { panic(err) }
    ...
}
```

Next we'll write a function that simulates a game, taking the number of players and marbles, and returning the player's score. I originally tried using a slice of ints to store the state of the game board, however this proved to be slow and inefficient with the number of insertions and deletions required. To improve performance we'll use a doubly linked list using the `container/list` package.

To move around our circle, the `container/list`, we'll use two helper functions. Normally you can use `list.Element.Next()` and `list.Element.Prev()` to move forward and backward through the list, however in our case we need the list to be circular, moving to the front when we reach the end, or vice versa. Our helper functions will take our list, the current element, and the number of movements to make, returning a new element. It will check after each Next() or Prev() if the returned element is nil, we move to the front, or back respectively.

```go
func CircPrev(l *list.List, p *list.Element, i int) *list.Element {
    for j := 0; j < i; j++ {
        p = p.Prev()
        if p == nil { p = l.Back() }
    }
    return p
}

func CircNext(l *list.List, p *list.Element, i int) *list.Element {
    for j := 0; j < i; j++ {
        p = p.Next()
        if p == nil { p = l.Front() }
    }
    return p
}
```

Our PlayGame function will initialize the player, PlayerScores and a list (l) with the first marble, number 0. We'll then iterate over our marbles, incrementing (circularly) the player number and checking if the current marble is a multiple of 23. If it is, it moves back, copies the element pointer and moves the element forward 1. Adds the element and the current marble to the player's score, and removes the specified element from the list. If the marble is not a multiple of 23, the current marble is inserted after the marble pointer and the pointer is moved forward. Once the last marble is placed, the game ends, and the map of player scores is returned.

```go
func PlayGame(Players, Marbles int) (PlayerScores map[int]int) {
    player := 0
    PlayerScores = map[int]int{}
    l := list.New()
    l.PushFront(0)
    p := l.Front()
    for i := 1; i <= Marbles; i++ {
        player += 1
        if player > Players { player = 1}

        if i % 23 == 0 {
            p = CircPrev(l, p, 8)
            r := p
            p = CircNext(l, p, 2)
            PlayerScores[player] += r.Value.(int) + i
            l.Remove(r)
            continue
        }
        l.InsertAfter(i,p)
        p = CircNext(l,p,2)
    }
    return
}
```

We can then find the highest score by simulating the game and iterating over the PlayerScores map, tracking the high score, getting our answer to part 1.

```go
    PlayerScores := PlayGame(players, marbles)
    P1Answer := 0
    for _, n := range PlayerScores {
        if n > P1Answer { P1Answer = n }
    }
    fmt.Printf("P1Answer: %d\n", P1Answer)
```

## Part 2

Part 2 asks us to simulate the same game again, with the small change being that there are now 100 times as many marbles. I originally implemented the PlayGame function with int slices, which was fast enough for the first part, however the second part would have taken dozens of hours. With the current implementation above, we simply feed in `marbles * 100` to our PlayGame function and the simulation completes in less than a second, giving us the answer for part 2.

```go
    PlayerScores = PlayGame(players, marbles * 100)
    P2Answer := 0
    for _, n := range PlayerScores {
        if n > P2Answer { P2Answer = n }
    }
    fmt.Printf("P2Answer: %d\n", P2Answer)
```

The complete code for this puzzle is available, along with the other days, on my Gitlab repository [https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_9/Day9.go](https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_9/Day9.go)

If you have any questions or comments, please feel free comment below.
