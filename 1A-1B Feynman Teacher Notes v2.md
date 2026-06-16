# Teacher Notes — Lessons 1A, 1A.5, and 1B

## Recommendation

Teach the handler before the Jetty server.

Do not start with `run-jetty`.

The student must first understand this sentence:

> A Ring handler is a function that takes a request map and returns a response map.

If he starts with Jetty first, too many things arrive at once:

- Leiningen project setup
- dependencies
- namespaces
- Jetty
- ports
- HTTP clients
- server process management
- handler behavior
- response maps

That is too much for a beginner.

The cleaner order is:

```text
1A: Handler as function
1A.5: Handler review gate
1B: Jetty calls the handler
```

---

## Why the attached handler note is useful

The attached note has a strong core idea:

> A Ring handler is just a function. Jetty is irrelevant when learning the handler itself.

That should be preserved.

Its best ideas to incorporate:

1. Call handlers directly before using Jetty.
2. Use fake request maps.
3. Show that a handler can ignore the request.
4. Show that a handler can later read from the request.
5. Use `{}` instead of `nil` for a fake request.
6. Explain `_` as an ignored parameter name by convention.
7. Treat Jetty as plumbing, not the main idea.

---

## Corrections made from the attached note

### 1. Fixed code formatting

The attached note had extra backticks at the end of code blocks, for example:

```clojure
(defn minimal-handler [_]
  {:status 200
   :headers {}
   :body ""})
```

### 2. Fixed function-name mismatch

The attached exercise defined:

```clojure
(defn request-type-handler [request] ...)
```

but then called:

```clojure
(practice1 ...)
```

The revised lesson uses matching names.

### 3. Corrected the `nil` explanation

The attached note says that reading `(:uri request)` will explode if `request` is `nil`.

That is not accurate in Clojure.

This returns `nil`:

```clojure
(:uri nil)
```

The better teaching point is:

> `nil` may not crash immediately, but it is not a request map and can hide weak tests.

So the student should use `{}` or a realistic fake request map.

### 4. Preserved the no-nonsense warning

The useful warning is kept:

> If the student cannot call the handler directly, he does not understand Ring yet.

But it is now used as a gate instead of a lecture.

---

## Lesson 1A teaching goal

The student should understand handlers without Jetty.

He should be able to say:

> I can call the handler myself because it is just a function.

He should know:

- `request` is input
- response map is output
- `:status`, `:headers`, and `:body` are response keys
- the handler can ignore the request
- the handler can read request keys such as `:uri` and `:request-method`

---

## Lesson 1A.5 teaching goal

This is a review gate before Jetty.

Use it if the student tends to copy commands quickly.

Require him to predict before running:

```clojure
(uri-handler {:uri "/"})
(uri-handler {:uri "/about"})
(uri-handler {})
```

The correct mental model:

> The output changes only when the handler reads data that changes.

---

## Lesson 1B teaching goal

Only after the student understands the handler, introduce Jetty.

The key sentence:

> Jetty receives real HTTP traffic, and the Ring Jetty adapter turns it into a request map before calling the handler.

The handler idea does not change.

What changes is who creates and passes the request map.

| Lesson | Who creates the request map? | Who calls the handler? |
|---|---|---|
| 1A | Student manually creates fake map | Student code / REPL |
| 1B | Ring Jetty adapter creates real map | Ring Jetty adapter |

---

## Questions to ask orally

Ask these before allowing the student to run Jetty:

1. Is a handler a server?
2. Can a handler be tested without a browser?
3. What is the difference between `{}` and `nil`?
4. What key stores the HTTP method in a Ring request map?
5. What key stores the URI?
6. What are the three common keys in a response map?
7. When Jetty is added, does the handler stop being a function?

Good answers:

1. No. A handler is a function. A server listens for requests.
2. Yes. Call the function directly with a map.
3. `{}` is an empty map; `nil` is not a map.
4. `:request-method`
5. `:uri`
6. `:status`, `:headers`, `:body`
7. No. It is still a function.

---

## Signs the student is copying

Watch for these:

- He can run `lein run` but cannot explain which function is called.
- He says Jetty creates the response body.
- He cannot explain what `request` is.
- He cannot predict output before running the handler.
- He thinks a browser is necessary to test a handler.
- He changes code randomly until it works.

Stop and return to Lesson 1A.5 if these appear.

---

## Minimal pass standard

Before Lesson 1B, he should be able to write this from memory:

```clojure
(defn handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "URI = " (:uri request))})
```

He should also be able to call it:

```clojure
(handler {:uri "/about"})
```

And explain the result in plain English.

---

## Teacher summary

Yes, handler should be learned before Jetty.

But do not overdo it.

The goal is not to master all of Ring before starting Jetty.

The goal is to make one thing solid:

> Handler = request map in, response map out.

Once that is solid, move to Jetty and show that Jetty simply supplies the real request map.
