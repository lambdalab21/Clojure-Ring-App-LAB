# 3D. Change Code Without Restarting Jetty

## Purpose

## 1. The idea

You started Jetty from the REPL.

Now Jetty is running.

In many workflows, a handler change means:

```text
edit code
stop server
restart server
test
```

In this workflow, a handler change can be:

```text
edit handler
evaluate handler
test
```

No server restart.

---

## 2. Change the handler

Change `my-handler` to:

```clojure
(defn my-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "Changed without restarting Jetty.\n"
              "URI: " (:uri request) "\n")})
```

Now evaluate only this function.

Place your cursor inside the `defn my-handler` form and run:

```text
C-M-x
```

Alternative:

```text
C-c C-k
```

`C-M-x` evaluates one top-level form.

`C-c C-k` reloads the whole buffer.

---

## 3. Test without restart

Do **not** run `(restart)`.

In the terminal:

```bash
curl -i http://localhost:8080/pizza
```

Expected body:

```text
Changed without restarting Jetty.
URI: /pizza
```

If that worked, you changed behavior in the running app.

### Check your understanding

1. Did Jetty restart? No. 
2. Did you run `lein run`? No. 
3. What did you evaluate? The updated my-handler definition. 
4. Why did the response change? Because #'my-handler' was passed to run-jetty, Jetty resolves the latest function value on each request. 

---

## 4. Why `#'my-handler` matters

In `start`, you used:

```clojure
(jetty/run-jetty #'my-handler
                 {:port 8080
                  :join? false})
```

`#'my-handler` means:

> Pass the Var for `my-handler`.

Feynman analogy:

> Passing `my-handler` is like giving Jetty a photocopy of the recipe. Passing `#'my-handler` is like telling Jetty, “Always read the current recipe from this page.”

That lets Jetty use the newer definition after you evaluate the handler again.

---

## 5. When restart is not needed

Usually no restart is needed when you change:

- text in the response body
- handler branching logic
- how the handler reads the request map
- helper functions called by the handler

---

## 6. When restart is needed

Usually restart is needed when you change server setup:

- port number
- Jetty options
- the function passed into `run-jetty`
- middleware wrapping done when the server starts

Example:

```clojure
{:port 8081
 :join? false}
```

If you change the port:

```text
C-c C-k
(restart)
curl new port
```

### Check your understanding

1. If you change only the body string, do you need restart? No.
2. If you change the port, do you need restart? Yes. 
3. If you change `:join? false` to another Jetty option, do you need restart? Yes. 
4. Why are handler logic and server setup different? The handler logic lives inside the re-evaluate function. Server setup is baked into the Jetty instance created at (start). 

---

## 7. Mini drill

Change the handler body to include the request method:

```clojure
(defn my-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "Changed without restarting Jetty.\n"
              "Method: " (:request-method request) "\n"
              "URI: " (:uri request) "\n")})
```

Evaluate only the function:

```text
C-M-x
```

Test:

```bash
curl -i http://localhost:8080/drill
```

Questions:

1. What changed? The response body now includes ht HTTP method. 
2. Did you restart Jetty? No. 
3. Which request-map keys did the handler read? :request-method and :uri.
4. Why is this faster than restarting the whole app? Re-evaluating one function 

---
