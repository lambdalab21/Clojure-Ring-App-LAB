# 0D. Middleware Preview — Save for Later

## Purpose

Do not teach this as part of the first Jetty lesson.

This file exists only to preserve the idea for later.

Middleware is important, but it should come **after** the student can already explain:

1. Jetty
2. Ring Jetty adapter
3. Ring request map
4. Ring handler
5. Ring response map

If those are shaky, middleware will become fake understanding.

---

## 1. The tiny preview

A Ring handler has this shape:

```text
request map → handler → response map
```

Middleware wraps a handler and returns another handler.

```text
request map → wrapper → original handler → response map
```

Feynman version:

> Middleware is like a helper standing next to the cook. The helper can look at the order, write something down, change the order, or check the tray before it leaves.

---

## 2. Example

```clojure
(defn wrap-debug [handler]
  (fn [request]
    (println "Request URI:" (:uri request))
    (handler request)))
```

Do not rush past this.

`wrap-debug` receives:

```clojure
handler
```

`wrap-debug` returns:

```clojure
(fn [request]
  ...)
```

That returned function is also a handler.

---

## 3. Questions for later

Do not use these questions until after the student can write and call handlers directly.

1. What does `wrap-debug` receive?
2. What does `wrap-debug` return?
3. Is the returned anonymous function also a handler?
4. Does Jetty need to know middleware exists?
5. Who prints the URI?
6. Who returns the final response?
7. Why does Ring middleware feel like function composition?

---

## 4. Teacher note

Introduce middleware later, after the student can answer this without hesitation:

> A handler is a function that receives a request map and returns a response map.

If that sentence is not automatic, middleware is too early.
