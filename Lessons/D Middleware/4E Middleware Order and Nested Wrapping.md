# 4E. Middleware Order and Nested Wrapping

## Purpose

This lesson answers one question:

> Why does middleware order matter?

Middleware is not decoration. Order changes behavior.

---

## 1. Current app

You may have:

```clojure
(def app
  (wrap-powered-by
    (wrap-log-request
      (wrap-add-student-name
        #'my-handler))))
```

This is nested wrapping.

---

## 2. Build order versus request order

Build order is easiest to read from inside out:

```text
#'my-handler
↓
wrap-add-student-name
↓
wrap-log-request
↓
wrap-powered-by
```

But request execution starts from the outside:

```text
request
↓
wrap-powered-by
↓
wrap-log-request
↓
wrap-add-student-name
↓
my-handler
```

Response comes back outward:

```text
my-handler
↓
wrap-add-student-name
↓
wrap-log-request
↓
wrap-powered-by
↓
client
```

Feynman version:

> The order ticket walks inward through the helpers to the cook. The finished tray walks back outward through the helpers.

---

## 3. Make order visible

Add these two middlewares:

```clojure
(defn wrap-enter-a [handler]
  (fn [request]
    (println "enter A")
    (let [response (handler request)]
      (println "leave A")
      response)))

(defn wrap-enter-b [handler]
  (fn [request]
    (println "enter B")
    (let [response (handler request)]
      (println "leave B")
      response)))
```

Now define:

```clojure
(def app
  (wrap-enter-a
    (wrap-enter-b
      #'my-handler)))
```

Reload, restart, and call:

```bash
curl http://localhost:8080/order
```

Expected printed order:

```text
enter A
enter B
leave B
leave A
```

---

## 4. Reverse the order

Now define:

```clojure
(def app
  (wrap-enter-b
    (wrap-enter-a
      #'my-handler)))
```

Reload, restart, and call:

```bash
curl http://localhost:8080/order
```

Expected printed order:

```text
enter B
enter A
leave A
leave B
```

### Stop and answer

1. Which middleware sees the request first in the first example?
2. Which middleware sees the response last in the first example?
3. What changed when you reversed the nesting?
4. Why is middleware order not harmless?

---

## 5. Use an analogy: entering and leaving rooms

Imagine the handler is inside a room.

Middleware A is an outer hallway.

Middleware B is an inner hallway.

To reach the handler:

```text
enter A → enter B → handler
```

To leave:

```text
handler → leave B → leave A
```

That is why logs often look like:

```text
enter A
enter B
leave B
leave A
```

---

## 6. Check your understanding

1. Does the outer middleware see the request before the inner middleware?
2. Does the outer middleware see the response after the inner middleware?
3. Why can middleware wrap both request and response behavior?
4. Why should you avoid adding many middlewares before understanding order?
5. What mistake happens if a student reads nested middleware only top-to-bottom?

---

## 7. Exit ticket

Explain this without looking:

> Middleware order matters because the request travels from the outer wrapper inward, but the response travels from the handler back outward through the wrappers.

If you cannot explain that, do not continue to the thread macro.
