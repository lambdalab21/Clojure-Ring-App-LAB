# 0C. First Coding Lab — See HTTP Become Ring Maps

## Purpose

Now you will create a tiny Clojure/Ring project and watch real HTTP requests become Ring request-map data.

This is the first coding lesson in this sequence.

Do not try to build a real web app yet. The goal is smaller:

> Send a URL with `curl`, then make the handler show what Ring put in the request map.

If you finish this lesson and only remember the commands, you failed. You must understand what each command proves.

---

## 1. What you should already know

Before starting, you should be able to say this from memory:

> Jetty receives and parses HTTP. The Ring Jetty adapter turns Jetty's Java request into a Ring request map. My handler receives that map and returns a response map.

And this:

> A handler is a function: request map in, response map out.

If you cannot say those clearly, go back to 0A and 0B.

---

## 2. Create the project

From the terminal:

```bash
lein new app http-map-demo
cd http-map-demo
```

Feynman version:

> `lein new app` creates a small Clojure application folder so we have a place to write and run code.

### Stop and answer

1. What folder did Leiningen create?
2. Why are we using a project instead of typing random code into a terminal?
3. What file usually lists project dependencies?

---

## 3. Add the Ring Jetty adapter dependency

Open `project.clj`.

Find the `:dependencies` section and make it look like this:

```clojure
:dependencies [[org.clojure/clojure "1.11.1"]
               [ring/ring-jetty-adapter "1.15.4"]]
```

Feynman version:

> Clojure is already the language. The Ring Jetty adapter is the library that lets our handler run inside Jetty.

Do not add extra libraries yet. More libraries create more places to hide confusion.

### Stop and answer

1. Which dependency is the Clojure language?
2. Which dependency lets us run a Ring handler with Jetty?
3. Why are we not adding routing or middleware libraries yet?

---

## 4. Replace the starter code

Open:

```text
src/http_map_demo/core.clj
```

Replace the file with this:

```clojure
(ns http-map-demo.core
  (:require [ring.adapter.jetty :as jetty])
  (:gen-class))

(defn request-summary [request]
  (str "Method: " (:request-method request) "\n"
       "URI: " (:uri request) "\n"
       "Query string: " (or (:query-string request) "nil") "\n"
       "User-Agent: " (or (get-in request [:headers "user-agent"]) "nil") "\n"))

(defn handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (request-summary request)})

(defn -main
  [& args]
  (jetty/run-jetty handler {:port 3000}))
```

### Why this code is intentionally plain

This is not the fanciest style. That is intentional.

We are separating the code into two simple jobs:

```clojure
request-summary
```

builds a string from the request map.

```clojure
handler
```

builds the Ring response map.

Feynman version:

> `request-summary` reads the order ticket. `handler` puts that summary onto the finished tray.

### Stop and answer

1. Which function receives the Ring request map?
2. Which function builds the response map?
3. What are the three keys in the response map?
4. Which function starts Jetty?
5. What port will Jetty listen on?

---

## 5. Predict before running

Before starting the server, predict the output for this request:

```bash
curl 'http://localhost:3000/search?q=clojure'
```

Fill this in:

```text
Method:
URI:
Query string:
User-Agent:
```

Do not skip this. Prediction is what turns typing into learning.

---

## 6. Run the app

From the project folder:

```bash
lein run
```

Keep this terminal open. The server is running there.

Open a second terminal and run:

```bash
curl 'http://localhost:3000/search?q=clojure'
```

Expected shape:

```text
Method: :get
URI: /search
Query string: q=clojure
User-Agent: curl/...
```

The exact user-agent may differ. That is fine.

### Compare

1. Did your predicted method match?
2. Did your predicted URI match?
3. Did your predicted query string match?
4. What surprised you?
5. What did this prove about the Ring request map?

---

## 7. Run more requests

Before each command, predict the `URI` and `Query string`.

```bash
curl 'http://localhost:3000/'
curl 'http://localhost:3000/about'
curl 'http://localhost:3000/todos?id=3&done=false'
curl -A 'StudentAgent' 'http://localhost:3000/agent-test'
```

### Questions

1. Which command had no query string?
2. Which command changed the user-agent?
3. Which part of the URL became `:uri`?
4. Which part became `:query-string`?
5. Did your handler parse raw HTTP text?

---

## 8. See the response map become an HTTP response

Stop the server with `Ctrl-c`.

Change the handler to this:

```clojure
(defn handler [request]
  {:status 404
   :headers {"Content-Type" "text/plain"}
   :body "The handler chose this 404."})
```

Run again:

```bash
lein run
```

In another terminal:

```bash
curl -i 'http://localhost:3000/anything'
```

You should see an HTTP response with status `404` and the body text.

### Stop and answer

1. Who chose the `404` in this example?
2. What did `curl -i` show that plain `curl` hides?
3. Which Ring response-map key became the HTTP status?
4. Which Ring response-map key became the HTTP body?
5. Which Ring response-map key became an HTTP header?

---

## 9. Restore the useful handler

Stop the server with `Ctrl-c`.

Restore the previous handler:

```clojure
(defn handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (request-summary request)})
```

Run it again:

```bash
lein run
```

Test one more time:

```bash
curl 'http://localhost:3000/final-check?lesson=0C'
```

---

## 10. Common mistakes

### Mistake 1: Typing the namespace path wrong

The namespace is:

```clojure
(ns http-map-demo.core
```

The file path is:

```text
src/http_map_demo/core.clj
```

Hyphen in namespace, underscore in file path. That is normal in Clojure.

---

### Mistake 2: Using `:header` instead of `:headers`

Wrong:

```clojure
:header {"Content-Type" "text/plain"}
```

Correct:

```clojure
:headers {"Content-Type" "text/plain"}
```

Ring expects `:headers`.

---

### Mistake 3: Thinking `lein run` is the REPL workflow

This lesson uses `lein run` because it is the simplest first coding lab.

Later you will learn the better Clojure workflow:

```text
start app from REPL → change code → evaluate code → refresh browser
```

Do not mix those ideas yet.

---

## 11. Final quiz

Answer from memory.

1. What command created the project?
2. What dependency let us run Jetty?
3. What function started Jetty?
4. What function handled the request?
5. What function built the response body string?
6. Which request-map key held `/search`?
7. Which request-map key held `q=clojure`?
8. Which response-map key became the HTTP status code?
9. Which response-map key became the HTTP body?
10. Why did we use `curl -i`?

---

## 12. Exit ticket

Close the file and explain this without looking:

> I created a Leiningen project, added the Ring Jetty adapter, wrote a handler, and ran Jetty. When `curl` sent an HTTP request, Jetty and the Ring adapter gave my handler a Ring request map. My handler returned a Ring response map. The adapter and Jetty turned that map into an HTTP response.

If you cannot explain that, you did not finish the lesson. You only typed commands.
