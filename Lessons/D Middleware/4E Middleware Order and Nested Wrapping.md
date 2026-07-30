# 4E. Middleware Order and Nested Wrapping
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

1. Which middleware sees the request first in the first example? wrap-enter-a
2. Which middleware sees the response last in the first example? wrap-enter-a
3. What changed when you reversed the nesting? Request and response travel order reversed
4. Why is middleware order not harmless? Order changes when code runs and can break logging, headers, auth, etc.

---


## 5. Check your understanding

1. Does the outer middleware see the request before the inner middleware? Yes. 
2. Does the outer middleware see the response after the inner middleware? Yes. 
3. Why can middleware wrap both request and response behavior? Each wrapper can run code before calling and next handler and after receiving the response. 
4. Why should you avoid adding many middlewares before understanding order? Wrong order produces surprising or broken behavior that is hard to debug. 
5. What mistake happens if a student reads nested middleware only top-to-bottom? They reverse the actual request/response flow. 
