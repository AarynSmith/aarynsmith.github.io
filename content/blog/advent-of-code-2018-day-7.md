---
title: "Advent of Code 2018 Day 7"
subtitle: ""
date: 2018-12-07
tags: ["AdventOfCode", "Go"]
draft: false
comments: true
---

It's that time of year again, Christmas is around the corner and Eric Wastl has gifted us with another set of Advent of Code puzzles.
<!--more-->
Welcome to [Advent Of Code 2018](https://adventofcode.com/2018/). You can read Part 1 of this series [Here](/blog/advent-of-code-2018-day-1/) or see all the parts of this series [Here](/tags/adventofcode/).

## Part 1

[Day 7](https://adventofcode.com/2018/day/7) gives us a list of tasks that must be completed before other tasks. Our goal is to find the correct order to process the tasks. As always, we'll need to begin with reading in our input file.

```go
package main

import (
    "fmt"
    "io/ioutil"
    "strings"
    "sort"
)

func main() {
    dat, err := ioutil.ReadFile("Day7.txt")
    if err != nil {
        panic(err)
    }
    input := strings.Split(string(dat), "\n")
    ...
}
```

We'll want a new struct to hold the information for our task. We'll call this struct Task, and it will contain the name of the step and a list of prerequisite steps.

```go
type Task struct {
    Name string
    Prereq []string
}
```

Next we'll need to loop through our list of tasks, creating our steps mapped by name and appending any prerequisites to the Prereq field.

```go
    Tasks := map[string]Task{}
    for _, s := range input {
        if len(s) < 36 {
            continue
        }
        pre, step := string(s[5]), string(s[36])

        st := Task{ Name: step,  }
        if _, ok := Tasks[step]; ok {
            st = Tasks[step]
        }
        if _, ok := Tasks[pre]; !ok {
            Tasks[pre] = Task{ Name: pre }
        }

        st.Prereq = append(st.Prereq, pre)

        Tasks[step] = st
    }
```

Next we'll need a couple of helper functions. The first, is a simple contains function to determine if a string slice contains a specified string.

```go
func contains(s []string, v string) bool {
    for _, k := range s {
        if k == v {
            return true
        }
    }
    return false
}
```

The next function will give us the steps that have either no prerequisites, or prerequisites that have already been completed. We can use these functions with a loop to determine the next task to complete.

```go
func AvailableTasks(Tasks map[string]Task, Complete []string) (Available []string) {
    for k := range Tasks {
        if contains(Complete, k) { continue }
        av := true
        for _, pre := range Tasks[k].Prereq {
            if !contains(Complete, pre){ av = false }
        }
        if av { Available = append(Available, k) }
    }
    sort.Strings(Available)
    return
}
```

Now that we can determine available tasks, we'll loop through finding all available tasks, and completing the first on the list alphabetically until there are no more available tasks. This gives us the sequence of tasks for the answer to our first part.

```go
    Complete := []string{}
    Available := AvailableTasks(Tasks, Complete)
    for len(Available) > 0 {
        Complete = append(Complete, Available[0])
        Available = AvailableTasks(Tasks, Complete)
    }
    fmt.Printf("P1Answer: %s\n", strings.Join(Complete, ""))
```

## Part 2

For part 2, things get a little more complicated. Instead of processing tasks single-threaded, we now have 5 workers, and instead of instantly completing tasks, the tasks have a processing time of 60 seconds + the value of the letter from 1 to 26. First we'll setup a struct for our Worker that will contain the current operation, as well as the remaining time on the operation.

```go
type Worker struct {
    CurrentOp Task
    TimeRemaining int
}
```

We can simulate the processing by setting up a pool of 5 workers, and letting them process the tasks as they become available. The main processing loop will iterate over the workers several times. The first time, any workers that have completed their task will mark them as completed. The next iteration through will assign the next available task to the workers, if any are available. Once all the tasks are assigned, or all the workers are processing something, we'll increment a "timer" variable, and decrement each worker's remaining time. Once all tasks are complete, our timer variable will have our answer to part 2.

```go
    Complete = []string{}
    Available =  []string{}
    S := 0
    Worker := []Task{
        Task{ CurrentOp: Task{}},
        Task{ CurrentOp: Task{}},
        Task{ CurrentOp: Task{}},
        Task{ CurrentOp: Task{}},
        Task{ CurrentOp: Task{}},
    }

    for len(Complete) < len(Tasks){
        for i := range Worker {
            if Worker[i].TimeRemaining == 0 && Worker[i].CurrentOp.Name != "" {
                Complete = append(Complete, Worker[i].CurrentOp.Name)
                Worker[i].CurrentOp = Task{}
            }
        }
        for i := range Worker {
            if Worker[i].CurrentOp.Name == "" {
                Available = NotInProgress(AvailableTasks(Tasks, Complete), Worker)
                if len(Available) > 0 {
                    Worker[i].CurrentOp, Available = Tasks[Available[0]], Available[1:]
                    Worker[i].TimeRemaining = int(Worker[i].CurrentOp.Name[0])-64 + 60
                }
             }
        }
        S += 1
        for i := range Worker {
            Worker[i].TimeRemaining -= 1
        }
    }
    fmt.Printf("P2Answer: %d\n", S-1)
```

The complete code for this puzzle is available, along with the other days, on my Gitlab repository [https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_7/Day7.go](https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_7/Day7.go)

If you have any questions or comments, please feel free comment below.
