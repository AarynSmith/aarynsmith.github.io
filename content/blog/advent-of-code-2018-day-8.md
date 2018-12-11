---
title: "Advent of Code 2018 Day 8"
subtitle: ""
date: 2018-12-08
tags: ["AdventOfCode", "Go"]
draft: false
comments: true
---

It's that time of year again, Christmas is around the corner and Eric Wastl has gifted us with another set of Advent of Code puzzles.
<!--more-->
Welcome to [Advent Of Code 2018](https://adventofcode.com/2018/). You can read Part 1 of this series [Here](/blog/advent-of-code-2018-day-1/) or see all the parts of this series [Here](/tags/adventofcode/).
 
## Part 1

[Day 8](https://adventofcode.com/2018/day/8) gives us a licence file from your navigation system. It consists of a series of numbers that describe a [tree](https://en.wikipedia.org/wiki/Tree_(data_structure)) of nodes. Each node consists of a header of two numbers, the number of child nodes, and number of metadata entries. Following the header is any children, then the metadata entries. To handle our tree, we'll start by creating a Node struct. Our node struct will have ints for the number of children and metadata entries, then a slice of Nodes, and a slice of ints for the metadata entries.

```go
type Node struct {
	NumChildren int
	NumMetadata int
	Children []Node
	Metadata []int
}
```

To begin, we'll read our licence file, trimming any whitespace from the end, and splitting the file on spaces, storing it in a slice called nums.

```go
func main(){
	input, err := ioutil.ReadFile("Day8.txt")
	if err != nil {
		panic(err)
	}
	nums := strings.Split(strings.TrimSpace(string(input)), " ")
    ...
}
```

Next we need to write a recursive function to parse a node, and it's children. We'll read the first two numbers off the list, and store them in our Node, then we'll iterate from 0 to the number of children, calling ParseChild again, and appending that to a slice of children, and updating our current position to the end of the parsed child. Lastly we'll read the metadata, update the position to the end of this entry and return the current node, and the last position of this node.

```go
func ParseChild(nums []string, pos int) (Node, int) {
	n := Node{}
	n.NumChildren, _ = strconv.Atoi(nums[pos+0])
	n.NumMetadata, _ = strconv.Atoi(nums[pos+1])
	pos += 2
	ch := []Node{}

	for j := 0; j < n.NumChildren; j++ {
		newCh, k := ParseChild(nums, pos)
		pos = k
		ch = append(ch, newCh)
	}
	n.Children = ch

	for _, m := range nums[pos: pos + n.NumMetadata] {
		meta, _ := strconv.Atoi(m)
		n.Metadata = append(n.Metadata, meta)
	}

	pos += n.NumMetadata
	return n, pos
}
```

Once we're able to parse our input and create the tree, we can write another short function that will take a node, and itterate recursively through it's children, then return a slice of ints containing the metadata for this node and all it's children.

```go
func GetChildMetadata(N Node) (meta []int) {
	for _, c := range N.Children {
		for _, m := range GetChildMetadata(c){
			meta = append(meta, m)
		}
	}
	for _, m := range N.Metadata {
		meta = append(meta, m)
	}
	return
}
```

We can use this function with a for loop to sum all the metadata, and get our answer to part 1 of the puzzle.

```go
	sum := 0
	for _, m := range GetChildMetadata(Tree){
		sum += m
    }
    
   	fmt.Printf("P1Answer: %d\n", sum)
```

## Part 2

Part 2 gives us an algorithm for calculating the value of a node based on the number of children it has. If there are no children, you simply sum the metadata entries, however, if there are child entries, the metadata entries become pointers to children of this node that need to be summed together if they exist. We can write another small function for finding the value of a node.

This function simply checks if a node has children, and if it does not returns the sum of the metadata. If the node does have children, it loops over the metadata checking if the child exists, and recursively gets the value of the child node, adding it to the running sum.

```go 
func GetChildValue(N Node) (v int){
	if len(N.Children)  == 0 {
		for _, c := range N.Metadata {
			v += c
		}
	} else {
		for _, m := range N.Metadata {
			if len(N.Children) > m - 1 {
				v += GetChildValue(N.Children[m - 1])
			}
		}
	}
	return
}
```

Running this function on our Root node, Tree, gives us the answer to the second part of the puzzle.

```go
	fmt.Printf("P2Answer: %d\n", GetChildValue(Tree))
```


The complete code for this puzzle is available, along with the other days, on my Gitlab repository [https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_8/Day8.go](https://gitlab.com/aarynsmith/adventofcode/blob/master/2018/Day_8/Day8.go)

If you have any questions or comments, please feel free comment below.
