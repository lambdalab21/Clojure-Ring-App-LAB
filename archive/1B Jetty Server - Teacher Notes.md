# Teacher Notes — Lesson 1A and 1B

## Why I added Lesson 1A

The original file moves too quickly from the conceptual Jetty/Ring explanation into project setup, dependencies, namespaces, `run-jetty`, curl, HTTPie, and process killing.

That is too much at once for a student who is still trying to understand what a handler is.

Lesson 1A adds one missing step:

> Call a handler manually before starting Jetty.

That makes Lesson 1B much easier:

> Jetty is not magic. Jetty and the Ring adapter simply call the same kind of handler with a real request map.

## Main changes from the original file

### 1. Fixed code issues

The original had several code problems that would confuse a beginner:

| Original issue | Fix |
|---|---|
| `:requre` typo | `:require` |
| `:require` placed outside the `ns` form | placed inside `(ns ...)` |
| `:header` key | `:headers` |
| Full namespace call everywhere | used alias: `[ring.adapter.jetty :as jetty]` then `jetty/run-jetty` |
| Dependency used broad `[ring "..."]` | used specific `[ring/ring-jetty-adapter "1.15.4"]` |
| Duplicate “What is a handler?” section | replaced with sequenced checks |

### 2. Added prediction before execution

The student must answer questions before running commands.

This blocks the bad pattern:

```text
copy → paste → run → done
```

The pattern should be:

```text
predict → run → observe → explain → modify
```

### 3. Used beginner-readable code

The code is not trying to be clever.

For example, the handler uses repeated response maps and simple `case` routing. That is not the most scalable style, but it is better for this lesson because the student can see exactly what is happening.

Do not introduce middleware, Compojure, Reitit, destructuring, or response helper libraries yet.

## Recommended teaching sequence

1. Previous file: `Understanding Jetty - Student Lesson.md`
2. New bridge lesson: `1A Handler First - Student Lesson.md`
3. Improved server lesson: `1B Jetty Server - Student Lesson.md`
4. Next lesson: manual routing deeper
5. Then introduce Compojure or another router
6. Then introduce middleware

## Answer guide — Lesson 1A

### Section 1

1. `lein new app jetty-demo`
2. Clojure namespace hyphens map to underscores in file paths.
3. `src/jetty_demo/core.clj`

### Section 2 prediction

1. `sample-request` is a Clojure map representing a fake request.
2. The input to `handler` is the request map.
3. The handler returns a response map.
4. No. There is no Jetty dependency and no `run-jetty` call.

### Section 3

1. No browser connected.
2. No Jetty request happened.
3. No Ring adapter created a map.
4. The map came from the code: `sample-request`.

### Section 4

1. The URI shown in the response body changed.
2. The handler reads `(:uri request)`.
3. `(:uri request)`.
4. No. The handler only needs a map with the expected keys.

### Section 5

1. Twice.
2. The request map is different.
3. The same function is called.
4. The input map.
5. The returned map.

### Exit ticket expected idea

A good answer should include:

- handler = ordinary Clojure function
- request map = data describing the incoming request
- response map = data describing what to send back
- manual call proves the function idea before server complexity
- in the next lesson, Ring Jetty adapter creates the request map

## Answer guide — Lesson 1B

### Section 2

1. `project.clj`
2. `[ring/ring-jetty-adapter "1.15.4"]`
3. `ring.adapter.jetty/run-jetty`
4. It teaches that only the Jetty adapter is needed for this lesson.

### Section 3

1. `lein deps`
2. `~/.m2/repository`
3. No.

### Section 4 prediction

1. `(jetty/run-jetty handler {:port 8080})`
2. `handler`
3. `{:port 8080}`
4. Server terminal should print method, URI, and query string.
5. Client should receive a text response.

### Section 6

1. `curl`, HTTPie, or browser.
2. Client terminal or browser.
3. Server terminal.
4. Jetty received the HTTP request, and the adapter called the handler.
5. No. The adapter created the request map.

### Section 7

1. The handler idea stayed the same.
2. The request map came from real HTTP traffic instead of hand-written data.
3. It removed mystery before adding Jetty.

### Section 8

1. `/`
2. `/about`
3. anything else, for example `/no-such-page`
4. `(:uri request)`
5. The handler manually chooses a response based on the URI.

### Section 9

1. Another process is already listening on that port.
2. `sudo lsof -i :8080`
3. Plain `kill` allows cleaner shutdown; `kill -9` force-kills.

## What to watch for

If the student says:

> Jetty returns the response.

Push back.

Better:

> The handler returns a Ring response map. The adapter converts that map into an HTTP response that Jetty writes back to the client.

If the student says:

> The request is the URL.

Push back.

Better:

> The URL is only part of the request. The request also includes method, headers, optional body, query string, and more.

If the student says:

> The server is the handler.

Push back.

Better:

> Jetty is the server. The handler is application logic.

## Suggested next lesson

Next lesson should not jump straight to Compojure yet.

First, make a lesson called:

```text
Lesson 1C — Manual Routing Before Compojure
```

Core idea:

```text
The router is just code that chooses which handler should handle the request.
```

Use this structure:

```clojure
(defn home-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body "Home\n"})

(defn about-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body "About\n"})

(defn not-found-handler [request]
  {:status 404
   :headers {"Content-Type" "text/plain"}
   :body "Not found\n"})

(defn app [request]
  (case (:uri request)
    "/" (home-handler request)
    "/about" (about-handler request)
    (not-found-handler request)))
```

This makes Compojure easier later because the student will understand what a routing library is replacing.
