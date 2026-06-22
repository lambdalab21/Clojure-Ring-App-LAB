# 4B. First Middleware — Logging the Request

## Purpose

This lesson answers one question:

> How can middleware do something before the handler runs?

You will write a middleware that prints the request URI.

No libraries. No routing. No thread macro yet.

---

## 1. Starting point

Your handler:

```clojure
(defn my-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "Hello from the REPL.\n"
              "URI: " (:uri request) "\n")})
```

The handler reads `:uri` and returns a response map.

---

## 2. Write `wrap-log-request`

Add this below `my-handler`:

```clojure
(defn wrap-log-request [handler]
  (fn [request]
    (println "Request URI:" (:uri request))
    (handler request)))
```

Feynman version:

> The helper looks at the order ticket, writes down the URI, then gives the same ticket to the cook.

---

## 3. Read the code carefully

```clojure
(defn wrap-log-request [handler]
```

This function receives the original handler.

```clojure
(fn [request]
```

It returns a new handler.

```clojure
(println "Request URI:" (:uri request))
```

Before calling the original handler, it prints the URI.

```clojure
(handler request)
```

Then it calls the original handler and returns whatever the handler returns.

---

## 4. Test by direct function call first

In the REPL:

```clojure
(def logged-handler
  (wrap-log-request my-handler))
```

Then call:

```clojure
(logged-handler {:uri "/abc"})
```

You should see printed output like:

```text
Request URI: /abc
```

And the function should return a response map.

### Stop and answer

1. Which function printed the URI?
2. Which function created the response map?
3. Did `wrap-log-request` change the response?
4. Is `logged-handler` a handler?
5. Why?

---

## 5. Use it with Jetty

Now change the running app.

Instead of running `my-handler` directly, define an app:

```clojure
(def app
  (wrap-log-request #'my-handler))
```

Then change `start`:

```clojure
(defn start []
  (if @server
    :server-already-running
    (do
      (reset! server
              (jetty/run-jetty #'app
                               {:port 8080
                                :join? false}))
      :server-started)))
```

Now reload the file:

```text
C-c C-k
```

Restart:

```clojure
(restart)
```

Then test:

```bash
curl -i http://localhost:8080/log-test
```

Look at two places:

1. The terminal or REPL where Jetty is running should print the log line.
2. The `curl` terminal should show the HTTP response.

---

## 6. Why `#'my-handler` inside `app`?

This line matters:

```clojure
(def app
  (wrap-log-request #'my-handler))
```

It passes a live reference to `my-handler`.

That helps the REPL workflow.

If you later redefine `my-handler`, the wrapped app can still call the current version.

Feynman version:

> The middleware helper is told to use the current recipe page, not a stale photocopy.

---

## 7. Check your understanding

Answer carefully.

1. What does `wrap-log-request` receive?
2. What does it return?
3. What does `app` become?
4. Why can Jetty run `app`?
5. Which function runs first: `wrap-log-request`'s returned function or `my-handler`?
6. Which function returns the final response map?
7. Why did we change Jetty from `#'my-handler` to `#'app`?

---

## 8. Exit ticket

Explain this without looking:

> `wrap-log-request` receives a handler and returns a new handler. The new handler prints the request URI, then calls the original handler. Jetty runs `app`, and `app` is just the wrapped handler.

If you cannot explain that, repeat this lesson.
