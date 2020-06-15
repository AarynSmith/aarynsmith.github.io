---
title: "AOC18: Repose Record"
subtitle: "Advent of Code 2018 Day 4"
date: 2018-12-04
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

[Day 4](https://adventofcode.com/2018/day/4) starts again with reading some data from a text file. Today we have a list of log entries for a Guard post. We have entries for when a guard comes on duty, falls asleep and wakes up again. Our task is to sort the list of entries by date and time, and find what guard sleeps the most.

Our first steps as always is read the file into a slice of lines.

```go
package main

import (
    "fmt"
    "io/ioutil"
    "strings"
    "time"
    "sort"
    "regexp"
    "strconv"
)

func main(){
    dat err := ioutil.ReadFile("Day4.txt")
    if err != nil {
        panic(err)
    }
    log := strings.Split(string(dat), "\n")
    ...
}
```

Next we need to have a way store and sort our entries, since the elves were so helpful to not store them in chronological (or any) order. We'll create a new data struct (logEntry), which will contain the parsed time of the entry, the raw entry text, the type of entry, the ID of the guard and the sleeping state of the entry. Just for fun, we'll also create a logType datatype and use some named consts using the iota function.

```go
type logType int

const (
    _ logType = iota
    startShift
    fallAsleep
    wakeUp
)

type logEntry struct {
    Time time.Time
    Entry string
    Type logType
    ID int
    State bool
}
```

Now, we'll create a slice of LogEntries, and iterate over the list from the file. For each line, we'll parse the time using time.Parse, and create our logEntry, filling just the Time and Entry. After sorting we'll be able to fill in the other information.

```go
    Log := []logEntry{}
    for _, entry := range log {
        if len(entry) < 17 {
            continue
        }
        t, err := time.Parse("2006-01-02 15:04", entry[1:17])
        if err != nil {
            panic(err)
        }
        Log = append(Log, logEntry{
            Time: t,
            Entry: entry,
        })
    }
```

To sort our slice we can utilize the sort package's sort.Slice method, all we need to do is provide a way for sort.Slice to determine if one item is less than the next. We can use the time.Time.Before to accomplish this.

```go
    sort.Slice(Log, func(i, j int) bool { return Log[i].Time.Before(Log[j].Time) })
```

Next we'll re-read our log entries in order, since the sleep and wake entries don't tell us the current guard ID, we'll have to track the last seen guard ID. We'll take a note from yesterday's puzzle and use a regular expression to find the guard ID from the begin shift entries. Once we have our information, we'll update the Log slice.

```go
    CurrentGuard := 0
    SleepState := false
    LogType := logType(0)
    GuardRegexp := regexp.MustCompile(`Guard #(\d+) begins shift`)
    for i, entry := range Log{
        switch t := entry.Entry[19:24]; t{
        case "Guard":
            m := GuardRegexp.FindStringSubmatch(entry.Entry)
            if m == nil {
                panic("Couldn't find Guard number")
            }
            CurrentGuard, _ = strconv.Atoi(m[1])
            LogType = StartShift
        case "falls":
            SleepState = true
            LogType = FallAsleep
        case "wakes":
            SleepState = false
            LogType = WakeUp
        default: panic("Unknown Status")
        }
        Log[i].ID = CurrentGuard
        Log[i].State = SleepState
        Log[i].Type = LogType
    }
```

Next we need a way to track the number of minutes each guard slept, as well as tallying the sleeps by minute. We'll define a new guardLog struct with a SleepTally integer, and a map of ints for tracking what minutes the guard slept.

```go
type guardLog struct {
    minuteTally map[int]int
    sleepTally int
}
```

We'll use this new struct in a map using the guard's ID as the key. Next we'll iterate over our log once again, tracking when the last fall asleep entry was, and when we get a wake up entry we'll either create a new guardLog, or use an existing guardLog if we've seen this guard already. We'll use a for loop to increment the by minute tally, then calculate the total amount of time slept. Then we'll store the new (or updated) guardLog back to our Guards map.

```go
    Guards := map[int]guardLog{}
    sleepTimeStart := 0
    for _, entry := range Log {
        if entry.Type == FallAsleep {
            sleepTimeStart = entry.Time.Minute()
        }
        if entry.Type == WakeUp {
            Gl := guardLog{    minuteTally: map[int]int{} }
            if _, o := Guards[entry.ID]; o{
                Gl = Guards[entry.ID]
            }
            for i := sleepTimeStart; i < entry.Time.Minute(); i++ {
                Gl.minuteTally[i]++
            }
            Gl.sleepTally += entry.Time.Minute() - sleepTimeStart
            Guards[entry.ID] = Gl
        }
```

Now we can finally figure out which of the guards slept the most, and in which minute he slept the most. Multiplying those two values together gives us our answer for part 1.

```go
    Part1Count, P1Answer := 0, 0
    for k,v := range Guards {
        maxMin, maxMinCount := 0, 0
        for i := 0; i < 60; i++ {
            if v.minuteTally[i] > maxMinCount {
                maxMin, maxMinCount = i, v.minuteTally[i]
            }
        }
        if v.sleepTally > Part1Count {
            Part1Count, P1Answer = v.sleepTally, k * maxMin
        }
    }
    fmt.Printf("Part 1 Answer: %d\n", P1Answer)
```

## Part 2

The next part what guard slept the most frequently on the same minute. We can simply add a couple blocks of code to our last section to find this information, getting the answer for part 2.

```go
    Part1Count, P1Answer := 0, 0
    Part2Count, P2Answer := 0, 0
    for k,v := range Guards {
        maxMin, maxMinCount := 0, 0
        for i := 0; i < 60; i++ {
            if v.minuteTally[i] > maxMinCount {
                maxMin, maxMinCount = i, v.minuteTally[i]
            }
            if v.minuteTally[i] > Part2Count {
                Part2Count, P2Answer = v.minuteTally[i], k * i
            }
        }
        if v.sleepTally > Part1Count {
            Part1Count, P1Answer = v.sleepTally, k * maxMin
        }
    }
    fmt.Printf("Part 1 Answer: %d\nPart 2 Answer: %d", P1Answer, P2Answer)
```

The complete code for this puzzle is available, along with the other days, on my Gitlab repository [https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_4/Day4.go](https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_4/Day4.go)

If you have any questions or comments, please feel free comment below.
