# End-of-Chapter Test — Version C
## Chapter: Jetty, Ring Maps, and Handler Simulation

### Instructions for the student

This version is code-reading heavy.

No typing into a REPL until after the test is finished.

---

## Part 1 — Vocabulary precision

Define each term in one sentence.

1. Jetty:
2. Ring Jetty adapter:
3. Ring request map:
4. Ring handler:
5. Ring response map:
6. HTTP response:

Your definitions must be simple enough for a younger student.

---

## Part 2 — Draw the flow

Draw the full request/response path.

You must include these items in the correct order:

- curl/browser
- Jetty
- HTTP parsing
- Java servlet request object
- Ring Jetty adapter
- Ring request map
- handler
- Ring response map
- HTTP response

Write your flow here:

```text









```

---

## Part 3 — Predict request-map pieces

For each URL, write `:uri` and `:query-string`.

### A

```text
http://localhost:3000/search?q=clojure
```

`:uri`:

`:query-string`:

---

### B

```text
http://localhost:3000/users/42?tab=settings
```

`:uri`:

`:query-string`:

---

### C

```text
http://localhost:3000/about
```

`:uri`:

`:query-string`:

---

## Part 4 — Handler result prediction

Use this handler:

```clojure
(defn handler [request]
  (case (:uri request)
    "/"      {:status 200
              :headers {"Content-Type" "text/plain"}
              :body "Home"}

    "/about" {:status 200
              :headers {"Content-Type" "text/plain"}
              :body "About"}

    {:status 404
     :headers {"Content-Type" "text/plain"}
     :body "Not found"}))
```

### A

```clojure
(handler {:request-method :get
          :uri "/about"
          :query-string "x=1"
          :headers {}})
```

What is the returned map?

```clojure

```

---

### B

```clojure
(handler {:request-method :get
          :uri "/contact"
          :query-string nil
          :headers {}})
```

What is the returned map?

```clojure

```

---

### C

```clojure
(handler {:request-method :post
          :uri "/"
          :query-string nil
          :headers {}})
```

What is the returned map?

```clojure

```

Important: Does this handler care about `:request-method`?

---

## Part 5 — Response map inspection

Given:

```clojure
{:status 500
 :headers {"Content-Type" "text/plain"}
 :body "Server error"}
```

Answer:

1. Which key becomes the HTTP status?
2. Which key becomes HTTP headers?
3. Which key becomes the response content?
4. Is this response map raw HTTP text?
5. Who turns this map into an actual HTTP response?

---

## Part 6 — Explain the dangerous beginner mistake

Explain why this is wrong:

> Since the URL is `/todos?id=3`, the Ring `:uri` must be `"/todos?id=3"`.

Correct explanation:

---

## Part 7 — Final Feynman answer

Explain the entire chapter in 6-8 sentences.

Your explanation must include:

- outside world
- inside Clojure data
- Jetty
- adapter
- handler
- 404 can come from the handler
