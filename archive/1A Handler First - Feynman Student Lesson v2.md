# Lesson 1A — Handler First: A Ring Handler Is Just a Function

## Simple idea first

Imagine a food counter.

A customer gives a worker an **order ticket**.

The worker reads the ticket and gives back a **tray**.

That is the basic idea of a Ring handler.

```text
order ticket  ->  worker   ->  tray
request map   ->  handler  ->  response map
```

Do not think about Jetty yet.

Do not think about browsers yet.

Do not think about ports yet.

In this lesson, you will call the handler yourself.

That means you will hand the function a pretend request map and inspect the response map it returns.

If you cannot do that, Jetty will only hide your confusion.

---

## The one sentence to own

Say this out loud:

> A Ring handler is a function that takes a request map and returns a response map.

Now say it like you are explaining it to a 10-year-old:

> A handler is a worker. You give it an order ticket. It gives you back a tray.

Both are the same idea.

| Simple word | Clojure/Ring word |
|---|---|
| order ticket | request map |
| worker | handler function |
| tray | response map |

---

## Goal

By the end, you should be able to answer these without looking:

1. What is a handler?
2. What does a handler receive?
3. What does a handler return?
4. Who calls the handler in this lesson?
5. Why are we not using Jetty yet?

Expected main idea:

> In this lesson, we call the handler ourselves so we can understand the function before adding the server.

---

## 1. Create a project

```bash
lein new app jetty-demo
cd jetty-demo
```

Leiningen creates a project.

The main file is:

```text
src/jetty_demo/core.clj
```

Notice the naming rule:

| Thing | Name |
|---|---|
| Project name | `jetty-demo` |
| Namespace | `jetty-demo.core` |
| File path | `src/jetty_demo/core.clj` |

Clojure namespaces use hyphens.

File paths use underscores.

### Check before moving on

Answer in your notes:

1. What command created the project?
2. What directory did you move into?
3. What file are you going to edit?
4. Why is the folder named `jetty_demo` but the namespace is `jetty-demo.core`?

---

## 2. First tiny handler

Open:

```text
src/jetty_demo/core.clj
```

Replace the file with this:

```clojure
(ns jetty-demo.core
  (:gen-class))

(defn minimal-handler [_]
  {:status 200
   :headers {}
   :body ""})

(defn -main
  [& args]
  (println "Calling minimal-handler directly.")
  (println (minimal-handler {})))
```

### Explain it simply

```clojure
(defn minimal-handler [_]
  ...)
```

Simple version:

> This function receives an order ticket, but it ignores the ticket.

Technical version:

> `_` is a parameter name that signals, “this argument is intentionally unused.”

This does not mean Clojure treats `_` specially here. It is still a name. Programmers use it by convention to show that they are ignoring the value.

### Predict before running

Write your prediction:

1. Does this start a web server?
2. Does this use Jetty?
3. What argument is passed to `minimal-handler`?
4. Does the handler use that argument?
5. What map does the handler return?

Now run:

```bash
lein run
```

Expected idea:

```clojure
{:status 200, :headers {}, :body ""}
```

The formatting may differ. The map idea should be the same.

---

## 3. What is the smallest practical response map?

A practical Ring response map usually has these keys:

```clojure
{:status 200
 :headers {}
 :body ""}
```

| Key | Simple meaning | HTTP meaning |
|---|---|---|
| `:status` | Did it work? | HTTP status code |
| `:headers` | Labels about the tray | HTTP response headers |
| `:body` | Main content | Response body |

### Explain it to a 10-year-old

Write one sentence for each:

1. `:status`
2. `:headers`
3. `:body`

Example:

> `:body` is the message we send back.

Do not copy that. Write your own.

---

## 4. A handler that returns text

Replace the code with this:

```clojure
(ns jetty-demo.core
  (:gen-class))

(defn hello-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body "Hello from my handler."})

(defn -main
  [& args]
  (println "Calling hello-handler directly.")
  (println (hello-handler {})))
```

### Important point

The handler receives `request`, but it does not use it yet.

That means this works:

```clojure
(hello-handler {})
```

This also returns the same response:

```clojure
(hello-handler {:uri "/about"})
```

Why?

Because the function ignores the request.

### Predict before running

1. What request map is passed to `hello-handler`?
2. Does the response body depend on the request map?
3. What will the body be?
4. Why does the handler return the same response every time?

Run:

```bash
lein run
```

---

## 5. Call the handler from the REPL

Now use the REPL.

```bash
lein repl
```

At the REPL, load the namespace if needed:

```clojure
(require 'jetty-demo.core)
```

Call the handler directly:

```clojure
(jetty-demo.core/hello-handler {})
```

Now call it with a more realistic pretend request:

```clojure
(jetty-demo.core/hello-handler
  {:request-method :get
   :uri "/"
   :headers {"host" "localhost"}})
```

You should get the same response.

### Explain why

Write the answer:

> Both calls return the same response because...

A weak answer:

> Because Clojure works that way.

A better answer:

> Because the handler does not read anything from the request map.

---

## 6. Make the handler care about the request

Now change the handler so it reads `:uri` from the request map.

Replace the file with this:

```clojure
(ns jetty-demo.core
  (:gen-class))

(defn uri-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "You asked for URI: " (:uri request))})

(defn -main
  [& args]
  (println (uri-handler {:uri "/"}))
  (println (uri-handler {:uri "/about"}))
  (println (uri-handler {})))
```

### Predict before running

For each call, predict the `:body` value:

```clojure
(uri-handler {:uri "/"})
(uri-handler {:uri "/about"})
(uri-handler {})
```

Now run:

```bash
lein run
```

### What happened with `{}`?

When the request map does not contain `:uri`, this expression returns `nil`:

```clojure
(:uri {})
```

So the body becomes something like:

```text
You asked for URI: 
```

or:

```text
You asked for URI: nil
```

depending on how it is printed.

The lesson is not “empty maps are always good.”

The lesson is:

> A handler can only use request data that exists in the request map.

---

## 7. About `{}` and `nil`

Use `{}` when you want an empty pretend request.

```clojure
(uri-handler {})
```

Do not use `nil` as your normal fake request.

```clojure
(uri-handler nil)
```

Why?

Because `nil` is not a request map.

Important Clojure detail:

```clojure
(:uri nil)
```

returns `nil`. It may not immediately crash.

That makes `nil` dangerous for practice: it can hide a bad test.

A request should be represented as a map, even an empty one.

Good practice:

```clojure
(uri-handler {})
```

Better practice:

```clojure
(uri-handler {:request-method :get
              :uri "/"
              :headers {"host" "localhost"}})
```

### Check your understanding

1. Why is `{}` better than `nil` for a fake request?
2. Does `(:uri nil)` always crash?
3. Why can `nil` hide mistakes?
4. What kind of value does a Ring handler normally expect?

---

## 8. Request method handler

Now make a handler that behaves differently for GET and POST.

Replace the file with this:

```clojure
(ns jetty-demo.core
  (:gen-class))

(defn request-method-message [request]
  (case (:request-method request)
    :get "GET request"
    :post "POST request"
    "Unknown request method"))

(defn request-method-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (request-method-message request)})

(defn -main
  [& args]
  (println (request-method-handler {:request-method :get}))
  (println (request-method-handler {:request-method :post}))
  (println (request-method-handler {})))
```

### Why split it into two functions?

This is not the shortest code.

That is intentional.

For learning, shorter is not always better.

```clojure
request-method-message
```

answers:

> What message should this request method produce?

```clojure
request-method-handler
```

answers:

> How do I wrap that message in a Ring response map?

This makes the ideas easier to see.

### Predict before running

Predict each response body:

1. `{:request-method :get}`
2. `{:request-method :post}`
3. `{}`
4. `{:request-method :delete}`

Now run:

```bash
lein run
```

Then try the fourth case in the REPL:

```clojure
(jetty-demo.core/request-method-handler {:request-method :delete})
```

---

## 9. Pretend what Jetty will do later

Later, Jetty and the Ring Jetty adapter will call your handler with a request map similar to this:

```clojure
{:request-method :get
 :uri "/index.html"
 :query-string nil
 :headers {"host" "localhost:8080"
           "user-agent" "curl/8.x"}
 :server-name "localhost"
 :server-port 8080
 :remote-addr "127.0.0.1"}
```

Simple explanation:

> Jetty receives a real HTTP request. The Ring adapter turns it into a Clojure map. Then your handler receives the map.

Nothing magical.

Still a function.

Still input map to output map.

---

## 10. Teach-back checkpoint

Close the notes.

Explain these out loud:

1. What is a Ring handler?
2. What is a request map?
3. What is a response map?
4. Why did we call the handler directly before using Jetty?
5. What does `_` mean as a parameter name?
6. Why is `{}` better than `nil` for practice?
7. What will Jetty add later?

If you cannot explain these simply, do not move to Jetty yet.

---

## Exit ticket

Write answers in complete sentences.

1. A handler is not a server. Explain why.
2. A handler can be tested without a browser. Explain how.
3. What does this return?

```clojure
(request-method-handler {:request-method :get})
```

4. What does this return?

```clojure
(request-method-handler {})
```

5. Explain this line to a 10-year-old:

```clojure
(defn request-method-handler [request] ...)
```

---

## Bottom line

A handler is the engine.

Jetty is the plumbing that brings real requests to the engine.

Learn the engine first.
