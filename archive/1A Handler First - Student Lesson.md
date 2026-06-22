# Lesson 1A — Before Jetty: A Ring Handler Is Just a Function

## Why this lesson exists

The previous lesson explained the idea:

> Jetty receives real HTTP traffic. The Ring adapter converts that traffic into a Clojure map. Your handler receives the map and returns another map.

Before starting Jetty, prove one thing first:

> A Ring handler is just a normal Clojure function.

If the student does not understand this, the next lesson becomes copy-paste theater.

---

## Goal

By the end of this lesson, you should be able to explain this sentence:

> A handler takes a request map and returns a response map.

You should also be able to call a handler yourself without starting a web server.

---

## 1. Create a small project

```bash
lein new app jetty-demo
cd jetty-demo
```

Leiningen creates a project like this:

```text
jetty-demo/
├── project.clj
└── src/
    └── jetty_demo/
        └── core.clj
```

Notice the small naming trap:

| Project name | Namespace | File path |
|---|---|---|
| `jetty-demo` | `jetty-demo.core` | `src/jetty_demo/core.clj` |

Clojure namespaces often use hyphens. File paths use underscores.

### Check your understanding

1. What command created the project?
2. Why is the folder named `jetty_demo` instead of `jetty-demo`?
3. Which file will you edit in this lesson?

---

## 2. Replace `src/jetty_demo/core.clj`

Open this file:

```text
src/jetty_demo/core.clj
```

Replace it with this code:

```clojure
(ns jetty-demo.core
  (:gen-class))

(def sample-request
  {:request-method :get
   :uri "/"
   :query-string nil})

(defn handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "The handler received this URI: " (:uri request) "\n")})

(defn -main
  [& args]
  (println "This lesson does not start Jetty yet.")
  (println "Calling the handler manually:")
  (println (handler sample-request)))
```

Do not run it yet.

### Predict before running

Write your prediction first.

1. What is the value of `sample-request`?
2. What is the input to `handler`?
3. What kind of value does `handler` return?
4. Will this start a web server? Why or why not?

---

## 3. Run the program

```bash
lein run
```

Expected idea, not exact formatting:

```text
This lesson does not start Jetty yet.
Calling the handler manually:
{:status 200, :headers {Content-Type text/plain}, :body The handler received this URI: /
}
```

The important part is not the formatting.

The important part is this:

```clojure
(handler sample-request)
```

That is just a normal function call.

### Check your understanding

1. Did a browser connect to this program?
2. Did Jetty receive an HTTP request?
3. Did the Ring adapter create a request map?
4. Where did the request map come from in this lesson?

Correct idea:

> The request map came from our own code, not from Jetty.

---

## 4. Change the fake request

Change `sample-request`:

```clojure
(def sample-request
  {:request-method :get
   :uri "/about"
   :query-string nil})
```

Run again:

```bash
lein run
```

### Questions

1. What changed in the output?
2. Why did the body change?
3. Which line of code reads the URI from the request map?
4. Does the handler care whether the map came from Jetty or from your own code?

---

## 5. Add one more request map

Change the code to this:

```clojure
(ns jetty-demo.core
  (:gen-class))

(def home-request
  {:request-method :get
   :uri "/"
   :query-string nil})

(def about-request
  {:request-method :get
   :uri "/about"
   :query-string nil})

(defn handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "The handler received this URI: " (:uri request) "\n")})

(defn -main
  [& args]
  (println "Home response:")
  (println (handler home-request))
  (println)
  (println "About response:")
  (println (handler about-request)))
```

Run it:

```bash
lein run
```

### Check your understanding

1. Is `handler` called once or twice?
2. What is different between the two calls?
3. What is the same between the two calls?
4. Which part is the request?
5. Which part is the response?

---

## 6. What this proves

This lesson proves the most important Ring idea:

```text
request map  ->  handler function  ->  response map
```

In the next lesson, Jetty will supply the request map for you.

But the handler idea does not change.

### Fill in the blanks

Complete this without looking above:

```text
A Ring handler is a __________.
It receives a __________ map.
It returns a __________ map.
Jetty does not replace the handler. Jetty helps call the __________.
```

---

## 7. Common beginner mistakes

### Mistake 1 — Thinking the handler is the server

Wrong:

> The handler is the server.

Better:

> The handler is the function that decides the response. Jetty is the server that listens for HTTP requests.

### Mistake 2 — Thinking the response body is the whole response

Wrong:

> The response is `"Hello"`.

Better:

> The response is a map with `:status`, `:headers`, and `:body`.

### Mistake 3 — Thinking Ring is required to write the handler

Wrong:

> I need Jetty before I can write a handler.

Better:

> I can write and test a handler as a normal Clojure function first.

---

## Exit ticket

Answer these in your own words:

1. What is a Ring handler?
2. What does the request map represent?
3. What does the response map represent?
4. Why did we call the handler manually before starting Jetty?
5. In the next lesson, who will create the request map?
