---
title: "Go REST API (Part 2)"
subtitle: Adding endpoints to a REST server
date: 2018-04-25
tags:  ["Go", "REST", "api"]
draft: false
comments: true
---

In this post we are going to expand on our REST API server enabling methods for our endpoints, and add the ability to Create and Delete users.
<!--more-->

## Methods for Endpoints

Now that we have a way to retrieve our user, we should restrict it to just the GET method. To do this we'll use the `mux.Router.Methods` function. This takes a list of methods we want this endpoint to respond to. We'll also go ahead and add our method for creating a new user also.

```go
func (a *App) Run(addr string) (err error) {
    log.Printf("Listening on %v", addr)
    a.Router.HandleFunc("/user/{id:[0-9]+}", a.GetUser).Methods("GET")
    a.Router.HandleFunc("/user", a.CreateUser).Methods("POST")
    return http.ListenAndServe(addr, a.Router)
}
```

## Creating users

As you can see we don't need to provide an ID for creating a user, as the system should provide the next available ID to us, using our new function `maxID`. We'll need to create our `app.CreateUser` function. Creating a user will be slightly more complicated than retrieving a user, mostly because we will need to get data from the client's body. To do this, we'll create a `bytes.Buffer` to read the `req.body` into, then use `json.Unmarshal` to convert it from a JSON string to a `user` object.

```go
import (
    "bytes"
)

func maxID(d map[int]user) (m int) {
    for k := range d {
       if k > m {
         m = k
       }
    }
    return m
}

func (a *app) CreateUser(w http.ResponseWriter, req *http.Request) {
    nextID := maxID(a.UserDB) + 1
    p := user{}
    buf := new(bytes.Buffer)
    buf.ReadFrom(req.Body)
    err := json.Unmarshal(buf.Bytes(), &p)
    if err != nil {
        w.WriteHeader(500)
        fmt.Fprintf(w, "500 Internal Server Error\n")
        return
    }
    p.ID = nextID
    a.UserDB[nextID] = p
    w.Header().Set("Location",
        fmt.Sprintf("http://%s/user/%d", req.Host, nextID))
    w.WriteHeader(200)
}
```

We see here that we are using the `w.Header().Set()` method to set a Location header which will tell the client the location of the newly created user. We also return the JSON formatted version of our new user.

Now to test this method, we  can't just simply request the page from our browser, because your browser sends a GET request. We'll need to send the request using something more specialized. For this test, we'll use curl:

```bash
$ curl -i -X POST http://127.0.0.1:3001/user -d'{"name": "User 2","role":"User"}'
HTTP/1.1 200 OK
Content-Length: 0
Content-Type: text/plain; charset=utf-8
Location: http://127.0.0.1:3001/user/2

```

This will create a second user in our database (or multiple if run more than once), and we can verify  our new user is created by running curl on the location returned above:

```bash
$ curl -i -X GET http://127.0.0.1:3001/user/2
HTTP/1.1 200 OK
Content-Length: 34
Content-Type: text/plain; charset=utf-8

{"ID":2,"Name":"User 2","Role":""}
```

## Deleting Users

Deleting users is a fairly easy process. We'll add a new endpoint for `user/{id}` for the DELETE method. Once we have the ID, we'll verify the ID exists in the database, and delete the key for that user from our map and return a 200. If the key does not exist, we'll return a 404.

```go
func (a *app) Run(addr string) (err error) {
    log.Printf("Listening on %v", addr)
    a.Router.HandleFunc("/user/{id:[0-9]+}", a.GetUser).Methods("GET")
    a.Router.HandleFunc("/user", a.CreateUser).Methods("POST")
    a.Router.HandleFunc("/user/{id:[0-9]+}", a.DeleteUser).Methods("DELETE")
    return http.ListenAndServe(addr, a.Router)
}

func (a *app) DeleteUser(w http.ResponseWriter, req *http.Request) {
    vars := mux.Vars(req)
    id, err := strconv.Atoi(vars["id"])
    if err != nil {
        w.WriteHeader(500)
        fmt.Fprintf(w, "Could not delete user: %v", err.Error())
        return
    }
    _, ok := a.UserDB[id]
    if !ok {
        w.WriteHeader(404)
        fmt.Fprintf(w, "404 page not found\n")
        return
    }
    delete(a.UserDB, id)
    w.WriteHeader(200)
}
```

Great! Let's test it out with curl again:

```bash
$ curl -i -X DELETE http://127.0.0.1:3001/user/1
HTTP/1.1 200 OK
Content-Length: 0
Content-Type: text/plain; charset=utf-8

```

## Conclusion

We've expanded on our server adding the ability add and remove users. Next we'll add the ability to Update by replacing using PUT and modify using PATCH.

Here is a gist of our [main.go](https://gist.github.com/AarynSmith/1f22c55d9d9ca4085a5e3aa50d846034#file-main-go) from this post.
