---
title: "Go REST API (Part 3)"
subtitle: More endpoints for REST Server
date: 2018-04-30

draft: false

tags:  ["Go", "REST", "api"]
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

In this post we are going to expand on our REST API server enabling methods for replacing a user using PUT and updating a user using both PATCH.
<!--more-->

## Replacing Users

We'll start off similar to our other methods by adding an new handler, and function for PUT. Replacing a user is going to be very similar to our `CreateUser` function, however we'll need to make sure the user exists before replacing it.

```go
func (a *App) Run(addr string) (err error) {
    log.Printf("Listening on %v", addr)
    a.Router.HandleFunc("/user/{id:[0-9]+}", a.GetUser).Methods("GET")
    a.Router.HandleFunc("/user", a.CreateUser).Methods("POST")
    a.Router.HandleFunc("/user/{id:[0-9]+}", a.ReplaceUser).Methods("PUT")
    return http.ListenAndServe(addr, a.Router)
}

func (a *app) ReplaceUser(w http.ResponseWriter, req *http.Request) {
    vars := mux.Vars(req)
    id, err := strconv.Atoi(vars["id"])
    if err != nil {
        w.WriteHeader(500)
        fmt.Fprintf(w, "Could not delete user: %v", err.Error())
        return
    }
    p := user{}
    buf := new(bytes.Buffer)
    buf.ReadFrom(req.Body)
    err = json.Unmarshal(buf.Bytes(), &p)
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
    p.ID = id
    a.UserDB[id] = p
    w.WriteHeader(200)
}
```

This is a simple method that takes the ID from the URL requested and assigns any incoming data to the user in the database. For updating a user with PATCH we'll have to check all the provided data in the request body and add them one by one.

Testing this method with curl is very similar to what we've seen previously, using the -X PUT flags.

```bash
$ curl -i -X PUT http://127.0.0.1:3001/user/1 -d'{"name": "User 2","role":"User"}'
HTTP/1.1 200 OK
Content-Length: 0
Content-Type: text/plain; charset=utf-8

$ curl -i -X GET http://127.0.0.1:3001/user/1
HTTP/1.1 200 OK
Content-Length: 34
Content-Type: text/plain; charset=utf-8

{"ID":1,"Name":"User 2","Role":"User"}
```

That being said, one potential issue with this method, is it by design will replace a user, not update it. So, if you pass only part of a user, such as a new name, it will not populate the role for the user. See below:

```bash
$ curl -i -X PUT http://127.0.0.1:3001/user/1 -d'{"name": "User 2"}'
HTTP/1.1 200 OK
Content-Length: 0
Content-Type: text/plain; charset=utf-8

$ curl -i -X GET http://127.0.0.1:3001/user/1
HTTP/1.1 200 OK
Content-Length: 34
Content-Type: text/plain; charset=utf-8

{"ID":1,"Name":"User 2","Role":""}
```

In order to only change part of a user instead of submitting all of the data for the user to be replaced, we want to use the PATCH method.

## Patching a user

Patching a user is generally more useful to update certain elements of a user rather than outright replacing the user in whole. For this we'll need to check the incoming request body data for the items that have been provided. We'll then update the current entry with the newly provided updates.

```go
func (a *app) Run(addr string) (err error) {
    log.Printf("Listening on %v", addr)
    a.Router.HandleFunc("/userdb", a.DumpDB).Methods("GET")
    a.Router.HandleFunc("/user/{id:[0-9]+}", a.GetUser).Methods("GET")
    a.Router.HandleFunc("/user", a.CreateUser).Methods("POST")
    a.Router.HandleFunc("/user/{id:[0-9]+}", a.DeleteUser).Methods("DELETE")
    a.Router.HandleFunc("/user/{id:[0-9]+}", a.ReplaceUser).Methods("PUT")
    a.Router.HandleFunc("/user/{id:[0-9]+}", a.UpdateUser).Methods("PATCH")
    return http.ListenAndServe(addr, a.Router)
}

func (a *app) UpdateUser(w http.ResponseWriter, req *http.Request) {
    vars := mux.Vars(req)
    id, err := strconv.Atoi(vars["id"])
    if err != nil {
        w.WriteHeader(500)
        fmt.Fprintf(w, "Could not delete user: %v", err.Error())
        return
    }
    pNew := user{}
    buf := new(bytes.Buffer)
    buf.ReadFrom(req.Body)
    err = json.Unmarshal(buf.Bytes(), &pNew)
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
    pNew.ID = id
    if pNew.Name == "" {
        pNew.Name = a.UserDB[id].Name
    }
    if pNew.Role == "" {
        pNew.Role = a.UserDB[id].Role
    }
    a.UserDB[id] = pNew
    w.WriteHeader(200)
}
```

Here you can see we unmarshal the incoming request body into an object called `pNew` and check to make sure the provided ID exists in the database. Once those have been verified, we check each of the struct fields from the incoming data to see if they have updated data, if they do not, they will be empty strings, since each of our struct fields are strings. If an incoming field is empty, we replace it with the data from the existing user information in the database, then overwrite the existing database entry with the updated one.

Let's try out our new method with curl using the -X PATCH flags.

```bash
$ curl -i -X GET http://127.0.0.1:3001/user/1
HTTP/1.1 200 OK
Content-Length: 39
Content-Type: text/plain; charset=utf-8

{"ID":1,"Name":"User 1","Role":"Admin"}

$ curl -i -X PATCH http://127.0.0.1:3001/user/1 -d'{"name": "User 2"}'
HTTP/1.1 200 OK
Content-Length: 0
Content-Type: text/plain; charset=utf-8

$ curl -i -X GET http://127.0.0.1:3001/user/1
HTTP/1.1 200 OK
Content-Length: 34
Content-Type: text/plain; charset=utf-8

{"ID":1,"Name":"User 2","Role":"Admin"}

$ curl -i -X PATCH http://127.0.0.1:3001/user/1 -d'{"role": "User"}'
HTTP/1.1 200 OK
Content-Length: 0
Content-Type: text/plain; charset=utf-8

$ curl -i -X GET http://127.0.0.1:3001/user/1
HTTP/1.1 200 OK
Content-Length: 38
Content-Type: text/plain; charset=utf-8

{"ID":1,"Name":"User 2","Role":"User"}
```

Here we see the first PATCH request changes user ID 1's name from "User 1" to "User 2" and the second request changes the user's role from "Admin" to "User".

## Conclusion

We've expanded on our server adding the ability replace and update users. Woo! We've got a mostly working API now. Next we'll add some tests to our project to verify all our API endpoints perform the way we expect them to work.

Here is a gist of our [main.go](https://gist.github.com/AarynSmith/2f09d3d2eace746e77e96c8f2ecf60c1#file-main-go) from this post.
