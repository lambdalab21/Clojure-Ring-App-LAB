# 3B. REPL-Controlled Server Code — Feynman Lesson

## Purpose

This lesson answers one question:

> What code do we need so the REPL can start and stop Jetty?

You are not using CIDER yet. First read and understand the server-control code.

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

Feynman version:

> The handler reads the request ticket and returns a response tray.

It receives a request map.

It returns a response map.

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

1. Is `my-handler` a normal function?
2. What does it receive?
3. What does it return?
4. Which request-map key does it read?
5. Which response-map key contains the text the client sees?

---

## 3. The server atom

```clojure
(defonce server (atom nil))
```

Feynman version:

> `server` is a small box. At first, the box is empty. Later, we put the running Jetty server object inside it.

Why do we need this?

Because if the REPL starts Jetty, the REPL also needs a way to stop Jetty later.

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

1. Why do we store the server object?
2. What does `@server` mean?
3. What does `reset!` do?
4. Why is `defonce` useful during REPL development?

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

Plain English:

> If a server is already stored in the box, do not start another one. If the box is empty, start Jetty, store the server object, and say `:server-started`.

Important pieces:

```clojure
:port 8080
```

means:

> Listen on port 8080.

```clojure
:join? false
```

means:

> Start Jetty but let the REPL keep accepting commands.

```clojure
#'my-handler
```

means:

> Give Jetty a live reference to the handler Var.

Feynman analogy:

> `my-handler` is like a photocopy of today's recipe. `#'my-handler` is like the recipe book page. If the recipe changes, Jetty can read the updated page.

### Check your understanding

1. Why does `start` check `@server` first?
2. What problem happens if two servers try to use the same port?
3. Why do we use `:join? false`?
4. Why do we use `#'my-handler`?

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

Plain English:

> If the server is running, stop it and empty the box. If there is no server, say so.

```clojure
(defn restart []
  (stop)
  (start))
```

Plain English:

> Stop, then start.

### Check your understanding

1. Why should `stop` reset the atom to `nil`?
2. What should happen if you call `(stop)` twice?
3. What does `(restart)` really do?
4. Why is `restart` not magic?

---

## 6. Exit ticket

Close the file and explain:

1. What the handler does.
2. What the server atom stores.
3. What `start` does.
4. What `stop` does.
5. Why `#'my-handler` matters.

If you cannot explain those, you are not ready to use CIDER yet.
