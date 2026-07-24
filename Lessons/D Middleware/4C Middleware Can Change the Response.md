# 4C. Middleware Can Change the Response

## 1. Review

This middleware does something before the handler:

```clojure
(defn wrap-log-request [handler]
  (fn [request]
    (println "Request URI:" (:uri request))
    (handler request)))
```

Flow:

```text
request map
↓
log middleware prints URI
↓
my-handler creates response map
↓
response map returns
```

---

## 2. Add a response-changing middleware

Add this:

```clojure
(defn wrap-powered-by [handler]
  (fn [request]
    (let [response (handler request)]
      (assoc-in response [:headers "X-Powered-By"] "Student Middleware"))))
```
---

## 3. Read the code slowly

```clojure
(defn wrap-powered-by [handler]
```

Receives a handler.

```clojure
(fn [request]
```

Returns a new handler.

```clojure
(let [response (handler request)]
```

Calls the original handler and stores its response map.

```clojure
(assoc-in response [:headers "X-Powered-By"] "Student Middleware")
```

Returns a modified response map with one extra response header.

---

## 4. Test directly first

In the REPL:

```clojure
(def powered-handler
  (wrap-powered-by my-handler))
```

Call it:

```clojure
(powered-handler {:uri "/power"})
```

Look at the returned response map.

You should see:

```clojure
:headers {"Content-Type" "text/plain"
          "X-Powered-By" "Student Middleware"}
```

### Stop and answer

1. Which function creates the original response? my-handler creates the original response. 
2. Which function adds the new header? wrap-powered-by adds the new header. 
3. Does `wrap-powered-by` change the request map? No. 
4. Does it change the response map? Yes. It changes the response map. 
5. Why is `assoc-in` used instead of `assoc`? assoc-in is used because :headers is a nested map. 

---

## 5. Wrap the app manually

For now, do not use the thread macro.

Define:

```clojure
(def app
  (wrap-powered-by
    (wrap-log-request
      #'my-handler)))
```

This is nested wrapping.

Read it from the inside out:

```text
#'my-handler
↓ wrapped by
wrap-log-request
↓ wrapped by
wrap-powered-by
```

The final result is still a handler.

---

## 6. Update and restart

Because `app` changed, reload the file:

```text
C-c C-k
```

Restart Jetty:

```clojure
(restart)
```

Test:

```bash
curl -i http://localhost:8080/powered
```

Look for:

```text
X-Powered-By: Student Middleware
```

---

## 7. Order of execution

With:

```clojure
(def app
  (wrap-powered-by
    (wrap-log-request
      #'my-handler)))
```

The request enters the outer middleware first:

```text
request
↓
wrap-powered-by returned function
↓
wrap-log-request returned function
↓
my-handler
↓
wrap-powered-by modifies response
↓
client receives response
```

Important:

- Request goes outside → inside.
- Response comes inside → outside.
---

## 8. Check your understanding

1. Is `app` still a handler? Yes. App is still a handler. 
2. Why can Jetty run `app`? Jetty can run ap because it's a valid ring handler. 
3. Which middleware is outside? wrap-powered-by is outside. 
4. Which middleware is inside? wrap-log-request is inside. 
5. Which middleware prints the URI? wrap-log-request prints the URI. 
6. Which middleware adds the response header? wrap-powered-by adds the response header.
7. Does the response pass back through the outer middleware? Yes. The response passes back through the outer middleware. 
8. Why does order matter? Order matters because requests flow inward and response flows outward affecting what each middleware can see our modify. 
