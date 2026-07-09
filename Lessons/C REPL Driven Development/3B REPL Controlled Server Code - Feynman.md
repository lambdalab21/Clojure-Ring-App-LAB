# 3B. REPL-Controlled Server Code

## Purpose

This lesson answers one question:

> What code do we need so the REPL can start and stop Jetty?


---

## 1. Replace the starter code

Open:

```text
src/sample/core.clj
```

Replace it with:

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

---

## 2. The handler

```clojure
(defn my-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "Hello from the REPL.\n"
              "URI: " (:uri request) "\n")})
```

Before Jetty, you should still be able to imagine this call:

```clojure
(my-handler {:uri "/test"})
```

Expected body:

```text
Hello from the REPL.
URI: /test
```

### Check your understanding

1. Is `my-handler` a normal function? Yes, my-handler is a normal Clojure function. 
2. What does it receive? It receives a request map. 
3. What does it return? It returns a response map. 
4. Which request-map key does it read? It reads the :uri key from the request map. 
5. Which response-map key contains the text the client sees? The :body key contains the text the client sees.  

---

## 3. The server atom

```clojure
(defonce server (atom nil))
```

```clojure
@server
```

means:

> Look inside the box.

```clojure
(reset! server something)
```

means:

> Replace what is inside the box.

`defonce` means:

> Create this box once. Do not erase it every time the file is reloaded.

### Check your understanding

1. Why do we store the server object? We store the server object so that we can stop it later. 
2. What does `@server` mean? @server dereferences the atom to get the current server for instance
3. What does `reset!` do? replaces the value inside of the atom. 
4. Why is `defonce` useful during REPL development? defonce prevents the atom from being reset when the namespace is reloaded in the REPL. 

---

## 4. The start function

```clojure
(defn start []
  (if @server
    :server-already-running
    (do
      (reset! server
              (jetty/run-jetty #'my-handler
                               {:port 8080
                                :join? false}))
      :server-started)))
```
```clojure
:port 8080
```

### Check your understanding
 
1. What problem happens if two servers try to use the same port? Two servers on the same port cause a "port already in use" error. 
2. Why do we use `:join? false`? :join? false lets the server run in the background so that the repl stays responsive. 
3. Why do we use `#'my-handler`? #'my-handler passes a var reference so that the code changes are picked up without restarting Jetty. 

---

## 5. The stop and restart functions

```clojure
(defn stop []
  (if @server
    (do
      (.stop @server)
      (reset! server nil)
      :server-stopped)
    :server-not-running))
```
```clojure
(defn restart []
  (stop)
  (start))
```
