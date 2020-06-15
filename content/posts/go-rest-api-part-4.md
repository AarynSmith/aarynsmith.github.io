---
title: "Go REST API (Part 4)"
subtitle: Testing REST server with Go Test
date: 2018-05-01

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

So we've built a working API server, but the only way we've tested the endpoints is by hammering them with `curl`, but why not use Go's built in testing mechanism, `go test`?
<!--more-->

## Test setup

We will start by creating our `main_test.go` file with a function for executing our request. This function will take an `http.Request` and a user database and respond with an `httptest.ResponseRecorder`. It will setup our server similar to the way our main function does, creating a new `app` instance, setting up the Router, and initializing the database.

```go
package main

import (
    "net/http"
    "net/http/httptest"

    "github.com/gorilla/mux"
)

func executeRequest(req *http.Request, db map[int]user) *httptest.ResponseRecorder {
    rr := httptest.NewRecorder()
    a := app{}
    a.Router = mux.NewRouter()
    a.UserDB = db
    a.Router.ServeHTTP(rr, req)
    return rr
}
```

## Adding Tests

We'll also need to write some tests. Let's start with testing our `app.GetUser` function. I'm a big fan of table driven testing. We'll setup a list of tests, and expected results and let Go iterate over the table checking our input and comparing it to our expected outcomes.

```go
func Test_app_GetUser(t *testing.T) {
    tests := []struct {
        database     map[int]user
        method       string
        request      string
        expectedCode int
    }{
        {
            database: map[int]user{
                1: user{
                    ID:   1,
                    Name: "User 1",
                    Role: "Admin",
                },
            },
            method:       "GET",
            request:      "/user/1",
            expectedCode: 200,
        },
    }
    for _, tt := range tests {
        req, _ := http.NewRequest(tt.method, tt.request, nil)
        response := executeRequest(req, tt.database)
        if tt.expectedCode != response.Code {
            t.Errorf("Expected response code %d. Got %d.\n", tt.expectedCode, response.Code)
        }
    }
}
```

Here we see we've setup a database with 1 user named "User 1" with a role of "Admin", and we are going to issue a GET request to our server to /user/1. We expect to get a 200 back. We tell the package to range over the `tests` variable and create a new `http.Request` using our `tests.method` and `tests.request` strings and run the request through our test method defined above with the `tests.database` user database. The response from that request is then compared to our `tests.expectedCode` and the test fails if it does not match.

So let's go ahead and run our tests:

```bash
$ go test
--- FAIL: Test_app_GetUser (0.00s)
        main_test.go:45: Expected response code 200. Got 404.
FAIL
exit status 1
FAIL    github.com/{username}/go-rest-api       0.004s
```

## Debugging Tests

Hmm... A 404 error. Why didn't we get the 200 response we expected? Let's look at our code. In our `main.go` we have the following:

```go
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

func (a *app) Run(addr string) (err error) {
    log.Printf("Listening on %v", addr)
    a.Router.HandleFunc("/user/{id:[0-9]+}", a.GetUser).Methods("GET")
    a.Router.HandleFunc("/user", a.CreateUser).Methods("POST")
    a.Router.HandleFunc("/user/{id:[0-9]+}", a.DeleteUser).Methods("DELETE")
    a.Router.HandleFunc("/user/{id:[0-9]+}", a.ReplaceUser).Methods("PUT")
    a.Router.HandleFunc("/user/{id:[0-9]+}", a.UpdateUser).Methods("PATCH")
    return http.ListenAndServe(addr, a.Router)
}
```

Our endpoint handles are defined in the Run function, but in our `main_test.go` we don't add those because the Run function actually spins the server up with the `http.ListenAndServe` call. Let's move our `Router.HandleFunc` definitions to a new function we can add to our  `executeRequest` function in `main_test.go`. Here is our updated `main.go`:

```go
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
    a.AddHandles()
    a.Run(":3001")
}

func (a *app) AddHandles() {
    a.Router.HandleFunc("/user/{id:[0-9]+}", a.GetUser).Methods("GET")
    a.Router.HandleFunc("/user", a.CreateUser).Methods("POST")
    a.Router.HandleFunc("/user/{id:[0-9]+}", a.DeleteUser).Methods("DELETE")
    a.Router.HandleFunc("/user/{id:[0-9]+}", a.ReplaceUser).Methods("PUT")
    a.Router.HandleFunc("/user/{id:[0-9]+}", a.UpdateUser).Methods("PATCH")
}

func (a *app) Run(addr string) (err error) {
    log.Printf("Listening on %v", addr)
    return http.ListenAndServe(addr, a.Router)
}
```

And here's the new version of our `executeRequest` function:

```go
func executeRequest(req *http.Request, db map[int]user) *httptest.ResponseRecorder {
    rr := httptest.NewRecorder()
    a := app{}
    a.Router = mux.NewRouter()
    a.UserDB = map[int]user{}
    a.AddHandles()
    a.Router.ServeHTTP(rr, req)
    return rr
}
```

Now let's re-run our tests and see if we get a pass:

```bash
$ go test
PASS
ok      github.com/{username}/go-rest-api      0.004s
```

Awesome. Our single test is passing. Now let's add some more tests.

## Testing unsuccessful requests

We'll write some more tests now for some of the ways requests might fail as well. What if we request a user that doesn't exist? It should respond with a 404:

```go
...
        {
            database: map[int]user{
                1: user{
                    ID:   1,
                    Name: "User 1",
                    Role: "Admin",
                },
            },
            method:       "GET",
            request:      "/user/2",
            expectedCode: 404,
        },
        {
            database:     map[int]user{},
            method:       "GET",
            request:      "/user/1",
            expectedCode: 404,
        },
    }
...
```

This adds tests for getting a 2nd user when only a 1 exists in the database, as well as getting user ID 1 when the database is empty. Another test we can write is what if an ID is provided in the URL that is too large to be represented as an integer? This should fail with a 500 error when `strcov.Atoi()` returns an error. We should probably test that as well. Let's add at test as follows:

```go
...
        {
            database:     map[int]user{},
            method:       "GET",
            request:      "/user/99999999999999999999",
            expectedCode: 500,
        },
...
```

Great, the only failure mode we haven't caught is if `json.Marshal` is unable to marshal our user struct, which should never happen since it's made up of basic data types like strings and integers.

## Tests, Tests, and More Tests

Now we can continue through replicating our `Test_app_GetUser` function for our other endpoint functions. First we'll start with `app.DeleteUser`. We'll do similar tests to the ones we wrote before. One test to delete an existing user, returning a 200. One test to delete a non-existent user, returning a 404. And finally, a test deleting a user with an overflowing ID, which will fail with a 500.

```go
func Test_app_DeleteUser(t *testing.T) {
    tests := []struct {
        database     map[int]user
        method       string
        request      string
        expectedCode int
    }{
        {
            database: map[int]user{
                1: user{
                    ID:   1,
                    Name: "User 1",
                    Role: "Admin",
                },
            },
            method:       "DELETE",
            request:      "/user/1",
            expectedCode: 200,
        },
        {
            database:     map[int]user{},
            method:       "DELETE",
            request:      "/user/1",
            expectedCode: 404,
        },
        {
            database:     map[int]user{},
            method:       "DELETE",
            request:      "/user/99999999999999999999",
            expectedCode: 500,
        },
    }
    for _, tt := range tests {
        req, _ := http.NewRequest(tt.method, tt.request, nil)
        response := executeRequest(req, tt.database)
        if tt.expectedCode != response.Code {
            t.Errorf("Expected response code %d. Got %d.\n", tt.expectedCode, response.Code)
        }
    }
}
```

For our other endpoints we'll have to add a few other options to our tests. Since `app.CreateUser`, `app.UpdateUser` and `app.ReplaceUser` all require data from the request body we'll have to provide that in our test as well.

## Tests with Bodies

We'll start with replicating our first curl test in a go test. For reference, our first curl request (from part 2) was `curl -i -X POST http://127.0.0.1:3001/user -d'{"name": "User 2","role":"User"}'`

We'll add a body field to our `tests` table, and assign it the value from our curl -d flag above. Then we can use that in our `http.NewRequest` in the third argument, which has been `nil` up until now. The problem is the `http.NewRequest` third argument takes an `io.Reader` not a string, so, we'll have to convert our body to an `io.Reader` using `string.NewReader` which takes a string and returns an `io.Reader`.

```go
func Test_app_CreateUser(t *testing.T) {
    tests := []struct {
        database     map[int]user
        method       string
        request      string
        body         string
        expectedCode int
    }{
        {
            database: map[int]user{
                1: user{
                    ID:   1,
                    Name: "User 1",
                    Role: "Admin",
                },
            },
            method:       "POST",
            request:      "/user",
            body:         `{"name": "User 2","role":"User"}`,
            expectedCode: 200,
        },
    }
    for _, tt := range tests {
        body := strings.NewReader(tt.body)
        req, _ := http.NewRequest(tt.method, tt.request, body)
        response := executeRequest(req, tt.database)
        fmt.Printf("%s %s: %+v\n", tt.method, tt.request, response.Body.String())
        if tt.expectedCode != response.Code {
            t.Errorf("Expected response code %d. Got %d.\n", tt.expectedCode, response.Code)
        }
    }
}
```

With this test we can also try to pass some invalid JSON input in the body. We'll add another test with some malformed JSON and we should get back a 500 server error, because `json.Unmarshal` should return an error.

```go
...
        {
            database: map[int]user{
                1: user{
                    ID:   1,
                    Name: "User 1",
                    Role: "Admin",
                },
            },
            method:       "POST",
            request:      "/user",
            body:         `Not a JSON string`,
            expectedCode: 500,
        },
...
```

Now if we run our tests:

```bash
$ go test
PASS
ok      github.com/AarynSmith/go-rest-api       0.005s
```

Awesome. Only thing left to test is Replacing and Updating users.

## Replacing user tests

Again very similar to our create user tests we'll create the tests for `app.ReplaceUser`, changing our method to PUT and the request to point to "/user/1".

```go
func Test_app_ReplaceUser(t *testing.T) {
    tests := []struct {
        database     map[int]user
        method       string
        request      string
        body         string
        expectedCode int
    }{
        {
            database: map[int]user{
                1: user{
                    ID:   1,
                    Name: "User 1",
                    Role: "Admin",
                },
            },
            method:       "PUT",
            request:      "/user/1",
            body:         `{"name": "User 2","role":"User"}`,
            expectedCode: 200,
        },
        {
            database: map[int]user{
                1: user{
                    ID:   1,
                    Name: "User 1",
                    Role: "Admin",
                },
            },
            method:       "PUT",
            request:      "/user/1",
            body:         `Not a JSON string`,
            expectedCode: 500,
        },
    }
    for _, tt := range tests {
        body := strings.NewReader(tt.body)
        req, _ := http.NewRequest(tt.method, tt.request, body)
        response := executeRequest(req, tt.database)
        if tt.expectedCode != response.Code {
            t.Errorf("Expected response code %d. Got %d.\n", tt.expectedCode, response.Code)
        }
    }
}
```

We'll also want to copy our tests from before with overflowing IDs, as well as writing a test for an ID that doesn't exist.

```go
        {
            database:     map[int]user{},
            method:       "PUT",
            request:      "/user/1",
            body:         `{"name": "User 2","role":"User"}`,
            expectedCode: 404,
        },
        {
            database:     map[int]user{},
            method:       "PUT",
            request:      "/user/99999999999999999999",
            body:         `{"name": "User 2","role":"User"}`,
            expectedCode: 500,
        },
```

And replicate the same tests for our the PATCH method, however we'll split the first test into two, one updating the name, and the second updating the role.

```go
func Test_app_UpdateUser(t *testing.T) {
    tests := []struct {
        database     map[int]user
        method       string
        request      string
        body         string
        expectedCode int
    }{
        {
            database: map[int]user{
                1: user{
                    ID:   1,
                    Name: "User 1",
                    Role: "Admin",
                },
            },
            method:       "PATCH",
            request:      "/user/1",
            body:         `{"name": "User 2"}`,
            expectedCode: 200,
        },
        {
            database: map[int]user{
                1: user{
                    ID:   1,
                    Name: "User 1",
                    Role: "Admin",
                },
            },
            method:       "PATCH",
            request:      "/user/1",
            body:         `{"role": "User"}`,
            expectedCode: 200,
        },
        {
            database:     map[int]user{},
            method:       "PATCH",
            request:      "/user/1",
            body:         `Not a JSON string`,
            expectedCode: 500,
        },
        {
            database:     map[int]user{},
            method:       "PATCH",
            request:      "/user/1",
            body:         `{"name": "User 2","role":"User"}`,
            expectedCode: 404,
        },
        {
            database:     map[int]user{},
            method:       "PATCH",
            request:      "/user/99999999999999999999",
            body:         `{"name": "User 2","role":"User"}`,
            expectedCode: 500,
        },
    }
    for _, tt := range tests {
        body := strings.NewReader(tt.body)
        req, _ := http.NewRequest(tt.method, tt.request, body)
        response := executeRequest(req, tt.database)
        if tt.expectedCode != response.Code {
            t.Errorf("Expected response code %d. Got %d.\n", tt.expectedCode, response.Code)
        }
    }
}
```

## Conclusion

At this point we can now run all the tests for our project and see what kind of code coverage we have.

```bash
$ go test --cover
PASS
coverage: 90.6% of statements
ok      github.com/{username}/go-rest-api       0.007s
```

Fantastic,we're covering 90% of our code. So we've built a REST API server in Go, setup endpoints and tested those endpoints. Have I missed anything? Feel free to send me a message from the links below.

Here is a gist of our [main.go](https://gist.github.com/AarynSmith/03dae2ec1c729891a13d6647cfb34155#file-main-go) and [main_test.go](https://gist.github.com/AarynSmith/03dae2ec1c729891a13d6647cfb34155#file-main_test-go) from this post.
