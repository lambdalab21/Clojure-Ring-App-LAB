# 4A. Middleware Is Just a Function — Feynman Lesson

## Purpose

This lesson answers one question:

> What is Ring middleware before any library gets involved?

Do **not** think about Hiccup, Reitit, reload, static files, or Ring-devel yet.

A middleware is not “a library feature.”

A middleware is just a function.

---

## 1. Starting code

Start from this code:

```clojure
(ns sample.core
  (:require [ring.adapter.jetty :as jetty]))

(defn my-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "Hello from the REPL.\n"
              "URI: " (:uri request) "\n")})

(defonce server (atom nil))

(defn start []
  (if @server
    :server-already-running
    (do
      (reset! server
              (jetty/run-jetty #'my-handler
                               {:port 8080
                                :join? false}))
      :server-started)))

(defn stop []
  (if @server
    (do
      (.stop @server)
      (reset! server nil)
      :server-stopped)
    :server-not-running))

(defn restart []
  (stop)
  (start))
```

At this point, Jetty runs `my-handler`.

```text
request map → my-handler → response map
```

---

## 2. Explain it to a 10-year-old

Imagine a cook in a restaurant.

The cook receives an order ticket and returns a tray of food.

Now imagine a helper standing next to the cook.

The helper can:

- look at the order before the cook sees it
- write something down
- change the order
- let the cook make the food
- inspect the tray before it goes out
- change the tray

That helper is middleware.

Feynman version:

> Middleware is a helper that wraps the cook. It receives the same order, calls the cook, and can look at or change the result.

Ring version:

```text
request map → middleware → handler → response map
```

More accurate:

```text
request map → wrapper function → original handler → response map
```

---

## 3. Handler shape

A handler has this shape:

```text
request map → response map
```

Example:

```clojure
(defn my-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body "Hello"})
```

You can call it directly:

```clojure
(my-handler {:uri "/test"})
```

---

## 4. Middleware shape

A middleware has this shape:

```text
handler → new handler
```

That is the sentence to memorize.

A middleware receives a handler and returns another handler.

```clojure
(defn wrap-something [handler]
  (fn [request]
    (handler request)))
```

This returned function is also a handler because it receives a request map and returns a response map.

---

## 5. The smallest useless middleware

This middleware does nothing useful:

```clojure
(defn wrap-do-nothing [handler]
  (fn [request]
    (handler request)))
```

But it teaches the shape.

Read it slowly:

```clojure
(defn wrap-do-nothing [handler]
```

Plain English:

> Make a function that receives a handler.

```clojure
(fn [request]
```

Plain English:

> Return a new function that receives a request.

```clojure
(handler request)
```

Plain English:

> Call the original handler with the request.

---

## 6. Test without Jetty

In the REPL:

```clojure
(def wrapped-handler
  (wrap-do-nothing my-handler))
```

Now call:

```clojure
(wrapped-handler {:uri "/test"})
```

You should get the same kind of response as calling:

```clojure
(my-handler {:uri "/test"})
```

### Stop and answer

1. What did `wrap-do-nothing` receive?
2. What did `wrap-do-nothing` return?
3. Is `wrapped-handler` also a handler?
4. Why?
5. What changed in the response?
6. Why is this still worth learning if it does nothing?

---

## 7. Important mental model

Do not say:

> Middleware is something Jetty does.

Better:

> Middleware is function wrapping around my handler.

Do not say:

> Middleware only comes from libraries.

Better:

> Libraries often provide middleware, but I can write middleware myself because middleware is just a function.

---

## 8. Exit ticket

Close the file and answer from memory:

1. A handler has the shape: __________ → __________.
2. Middleware has the shape: __________ → __________.
3. Middleware receives a __________.
4. Middleware returns a __________.
5. The returned function is a handler because it receives a __________ and returns a __________.

If you cannot answer these, do not continue.
