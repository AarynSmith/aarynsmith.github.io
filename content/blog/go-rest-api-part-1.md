---
title: "Go REST API (Part 1)"
subtitle: Building a REST API in Go
date: 2018-04-22
tags:  ["Go", "REST", "api"]
draft: false
comments: true
---

In this post we are going to learn how to build a REST server in Go.
<!--more-->

## What is REST

*REST* is an acronym for Representational State Transfer. It is an architectural style for facilitating communications between a stateless client and server. The five basic HTTP verbs used in a REST system are GET, POST, PUT, PATCH and DELETE.

* GET retrieves a specific resource or collection of resources.
* POST creates a new resource.
* PUT replaces an existing resource.
* PATCH updates an existing resource.
* DELETE removes a resource.

These methods generally make up what is referred to as a CRUD system (Create Read Update Delete).

## Getting Started

To start setting up our API we'll start with a new project in GO. Create a new project folder in your GOPATH:

```bash
mkdir -p $GOPATH/src/github.com/{username}/go-rest-api
cd $GOPATH/src/github.com/{username}/go-rest-api
```

We will also need to get our router package:

```bash
go get github.com/gorilla/mux
```

## Creating our server

Next we'll create our `main.go` file.

```go
package main

import (
    "fmt"
    "log"
    "net/http"

    "github.com/gorilla/mux"
)

type app struct {
    Router   *mux.Router
}

func (a *app) Run(addr string) (err error) {
    log.Printf("Listening on %v", addr)
    return http.ListenAndServe(addr, a.Router)
}

func main() {
    a := app{}
    a.Router = mux.NewRouter()
    a.Run(":3001")
}
```

Then run it with `go run main.go` and fire up a web browser pointed at [http://127.0.0.1:3001](http://127.0.0.1:3001) and see what we get.

```text
404 page not found
```

## Adding an endpoint

Uh oh...Looks like we forgot something. Well, we need to tell the server what to listen for. Let's add an endpoint. To do this we need to use the `mux.Router.HandleFunc` method, which takes a string for the path, and a `func(http.ResponseWriter, *http.Request)`. Lets add another function to `main.go`:

```go
func (a *app) Test(w http.ResponseWriter, req *http.Request) {
    fmt.Fprint(w, "Test Enpoint Called")
}

```

Then we'll tell the router to run this function when we send a request to an endpoint:

```go
func (a *app) Run(addr string) (err error) {
    log.Printf("Listening on %v", addr)
    a.Router.HandleFunc("/test", a.Test)
    return http.ListenAndServe(addr, a.Router)
}
```

Now when we launch the server with `go run main.go` and point our browser at [http://127.0.0.1:3001/test](http://127.0.0.1:3001/test) we see the following:

```text
Test Endpoint Called
```

## Working example

Great! We have a working test endpoint, so lets make it do something actually useful. We'll create a user management server. We'll be able to Create, Retrieve, Update, and Delete users.

First we need to add a "database" to store our users. For this example We'll create a struct `user` that will contain an individual user, and use an in-memory map of users. We'll also initialize the database with the first user "User 1" with an ID of 1 when we start the server.

```go
type app struct {
    Router *mux.Router
    UserDB map[int]user
}

type user struct {
    ID   int
    Name string
    Role string
}

func main() {
    a := app{}
    a.Router = mux.NewRouter()
    a.UserDB = map[int]user{
        1: user{
            ID:   1,
            Name: "User 1",
            Role: "Admin",
        },
    }
    a.Run(":3001")
}
```

Also, lets change our test endpoint to an endpoint called user:

```go
func (a *app) Run(addr string) (err error) {
    log.Printf("Listening on %v", addr)
    a.Router.HandleFunc("/user", a.GetUser)
    return http.ListenAndServe(addr, a.Router)
}

func (a *app) GetUser(w http.ResponseWriter, req *http.Request) {
    fmt.Fprintf(w, "Todo: return a user...\n")
}
```

Better, but we need to be able to specify a user ID for our request. How do we tell the server what user we want to return? Well, we need to add the ID to the endpoint HandleFunc, but at the same time, we want to make sure only valid IDs are passed to our endpoint since our ID above is defined as an `int`. To do this we use a regular expression after the variable identifier on the endpoint, and we can retrieve those variables from the request using `mux.Vars`

```go
func (a *app) Run(addr string) (err error) {
    log.Printf("Listening on %v", addr)
    a.Router.HandleFunc("/user/{id:[0-9]+}", a.GetUser)
    return http.ListenAndServe(addr, a.Router)
}
func (a *app) GetUser(w http.ResponseWriter, req *http.Request) {
    vars := mux.Vars(req)
    fmt.Fprintf(w, "Todo: return a user...%s\n", vars["id"])
}
```

And finally we'll get the User's name from the database and return it via the `http.ResponseWriter`. Since `mux.Vars` returns the ID as a string, we'll have to convert it to an `int`. If we are unable to convert the ID to an `int` we'll send back a 500 Internal Server Error, with the error message from `strconv.Atoi`. If the ID provided is not found we'll want to return a 404 error. We'll also want to encode our output in a format that is useful on many different remote systems. In this case we'll use JSON.

```go
import (
    "encoding/json"
)

func (a *app) GetUser(w http.ResponseWriter, req *http.Request) {
    vars := mux.Vars(req)
    id, err := strconv.Atoi(vars["id"])
    if err != nil {
        w.WriteHeader(500)
        fmt.Fprintf(w, "500 Internal Server Error\n")
        return
    }
    _, ok := a.UserDB[id]
    if !ok {
        w.WriteHeader(404)
        fmt.Fprintf(w, "404 page not found\n")
        return
    }
    j, err := json.Marshal(a.UserDB[id])
    if err != nil {
        w.WriteHeader(500)
        fmt.Fprintf(w, "500 Internal Server Error\n")
        return
    }
    fmt.Fprint(w, string(j))
}
```

## Conclusion

We've created a basic server that responds to a GET request from a browser to return a user. In the next part we will look at adding our basic methods to the user endpoint to let us Create, and Delete users.

Here is a gist of our [main.go](https://gist.github.com/AarynSmith/15cf2186d639ad0cf44afafa0591e40b#file-main-go) from this post.
