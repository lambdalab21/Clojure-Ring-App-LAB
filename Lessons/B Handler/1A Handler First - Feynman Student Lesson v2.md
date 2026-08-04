# Lesson 1A — Handler First: A Ring Handler Is Just a Function

## Goal

By the end, you should be able to answer these without looking:

1. What is a handler?
2. What does a handler receive?
3. What does a handler return?
4. Who calls the handler in this lesson?
5. Why are we not using Jetty yet?

---

## 1. Create a project

```bash
lein new app jetty-demo
cd jetty-demo
```

Leiningen creates a project.

The main file is:

```text
src/jetty_demo/core.clj
```

Notice the naming rule:

| Thing | Name |
|---|---|
| Project name | `jetty-demo` |
| Namespace | `jetty-demo.core` |
| File path | `src/jetty_demo/core.clj` |

Clojure namespaces use hyphens.

File paths use underscores.

### Check before moving on

Answer in your notes:

1. What command created the project?
2. What directory did you move into?
3. What file are you going to edit?
4. Why is the folder named `jetty_demo` but the namespace is `jetty-demo.core`?

---

## 2. First handler

Open:

```text
src/jetty_demo/core.clj
```

Replace the file with this:

```clojure
(ns jetty-demo.core
  (:gen-class))

(defn minimal-handler [_]
  {:status 200
   :headers {}
   :body ""})

(defn -main
  [& args]
  (println "Calling minimal-handler directly.")
  (println (minimal-handler {})))
```

---

## 3. What is the smallest practical response map?

A practical Ring response map usually has these keys:

```clojure
{:status 200
 :headers {}
 :body ""}
```

| Key | Simple meaning | HTTP meaning |
|---|---|---|
| `:status` | Did it work? | HTTP status code |
| `:headers` | Labels about the tray | HTTP response headers |
| `:body` | Main content | Response body |

---

## 4. A handler that returns text

Replace the code with this:

```clojure
(ns jetty-demo.core
  (:gen-class))

(defn hello-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body "Hello from my handler."})

(defn -main
  [& args]
  (println "Calling hello-handler directly.")
  (println (hello-handler {})))
```

---

## 5. Call the handler from the REPL

Now use the REPL.

```bash
lein repl
```

At the REPL, load the namespace if needed:

```clojure
(require 'jetty-demo.core)
```

Call the handler directly:

```clojure
(jetty-demo.core/hello-handler {})
```

Now call it with a more realistic pretend request:

```clojure
(jetty-demo.core/hello-handler
  {:request-method :get
   :uri "/"
   :headers {"host" "localhost"}})
```

---

## 6. Make the handler care about the request

Now change the handler so it reads `:uri` from the request map.

Replace the file with this:

```clojure
(ns jetty-demo.core
  (:gen-class))

(defn uri-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "You asked for URI: " (:uri request))})

(defn -main
  [& args]
  (println (uri-handler {:uri "/"}))
  (println (uri-handler {:uri "/about"}))
  (println (uri-handler {})))
```

### Predict before running

For each call, predict the `:body` value:

```clojure
(uri-handler {:uri "/"})
(uri-handler {:uri "/about"})
(uri-handler {})
```

Now run:

```bash
lein run
```

### What happened with `{}`?

When the request map does not contain `:uri`, this expression returns `nil`:

```clojure
(:uri {})
```

So the body becomes something like:

```text
You asked for URI: 
```

or:

```text
You asked for URI: nil
```

depending on how it is printed.

---

## 7. About `{}` and `nil`

Use `{}` when you want an empty pretend request.

```clojure
(uri-handler {})
```

Do not use `nil` as your normal fake request.

```clojure
(uri-handler nil)
```

### Check your understanding

1. Why is `{}` better than `nil` for a fake request?
2. Does `(:uri nil)` always crash?
3. Why can `nil` hide mistakes?
4. What kind of value does a Ring handler normally expect?

---

## 8. Request method handler

Now make a handler that behaves differently for GET and POST.

Replace the file with this:

```clojure
(ns jetty-demo.core
  (:gen-class))

(defn request-method-message [request]
  (case (:request-method request)
    :get "GET request"
    :post "POST request"
    "Unknown request method"))

(defn request-method-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (request-method-message request)})

(defn -main
  [& args]
  (println (request-method-handler {:request-method :get}))
  (println (request-method-handler {:request-method :post}))
  (println (request-method-handler {})))
```

---

## 9. Pretend what Jetty will do later

Later, Jetty and the Ring Jetty adapter will call your handler with a request map similar to this:

```clojure
{:request-method :get
 :uri "/index.html"
 :query-string nil
 :headers {"host" "localhost:8080"
           "user-agent" "curl/8.x"}
 :server-name "localhost"
 :server-port 8080
 :remote-addr "127.0.0.1"}
```

---

## 10. Teach-back checkpoint

Explain these:

1. What is a Ring handler?
2. What is a request map?
3. What is a response map?
4. Why did we call the handler directly before using Jetty?
5. What does `_` mean as a parameter name?
6. Why is `{}` better than `nil` for practice?
7. What will Jetty add later?

---

## Exit ticket

1. A handler is not a server. Explain why.
2. A handler can be tested without a browser. Explain how.
4. What does this return?

```clojure
(request-method-handler {})
```
