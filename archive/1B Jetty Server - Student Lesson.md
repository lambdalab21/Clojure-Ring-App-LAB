# Lesson 1B — Start a Jetty Server with a Ring Handler

## Where this lesson fits

In Lesson 1A, you called a Ring handler manually:

```text
request map  ->  handler function  ->  response map
```

In this lesson, Jetty will receive a real HTTP request, and the Ring Jetty adapter will call the same kind of handler for you.

The idea does not change.

The source of the request map changes.

---

## Goal

By the end of this lesson, you should be able to explain this flow:

```text
browser / curl / httpie
   ↓
Jetty server
   ↓
Ring Jetty adapter
   ↓
handler receives request map
   ↓
handler returns response map
   ↓
client receives HTTP response
```

You should also be able to identify which part of the code starts the server and which part handles requests.

---

## 1. Continue from the same project

Use the same project from Lesson 1A:

```bash
cd jetty-demo
```

If you did not do Lesson 1A, stop. Do that first. Otherwise this lesson will feel like magic.

---

## 2. Add the Jetty adapter dependency

Open `project.clj`.

Find the `:dependencies` section.

Keep the Clojure version that Leiningen generated. Add the Jetty adapter line.

Example:

```clojure
:dependencies [[org.clojure/clojure "1.11.1"]
               [ring/ring-jetty-adapter "1.15.4"]]
```

Your Clojure version may be newer. That is fine. The important new line is:

```clojure
[ring/ring-jetty-adapter "1.15.4"]
```

This dependency gives your project access to:

```clojure
ring.adapter.jetty/run-jetty
```

The dependency name is specific on purpose:

```clojure
[ring/ring-jetty-adapter "1.15.4"]
```

It says: “I need the Ring adapter that runs Jetty.”

### Check your understanding

1. Which file did you edit?
2. What dependency did you add?
3. What library function do we want from that dependency?
4. Why is this dependency more specific than adding all of `[ring "..."]`?

---

## 3. Download dependencies

Run:

```bash
lein deps
```

Leiningen downloads dependencies into your local Maven repository:

```text
~/.m2/repository
```

You usually do not edit files there. Leiningen manages them.

### Check your understanding

1. What command downloaded the dependency files?
2. Where are dependency files stored on your machine?
3. Should you manually edit files inside `~/.m2/repository`?

---

## 4. Replace `src/jetty_demo/core.clj`

Use this code:

```clojure
(ns jetty-demo.core
  (:require [ring.adapter.jetty :as jetty])
  (:gen-class))

(defn handler [request]
  (println "REQUEST METHOD:" (:request-method request))
  (println "URI:" (:uri request))
  (println "QUERY STRING:" (:query-string request))
  (println)
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "Hello from Jetty + Ring.\n"
              "You requested: " (name (:request-method request)) " " (:uri request) "\n")})

(defn -main
  [& args]
  (println "Starting server on http://localhost:8080")
  (jetty/run-jetty handler {:port 8080}))
```

### Important code reading

Do not skip this part.

```clojure
(:require [ring.adapter.jetty :as jetty])
```

This loads the Jetty adapter namespace and gives it the short name `jetty`.

```clojure
(jetty/run-jetty handler {:port 8080})
```

This starts Jetty and tells it two things:

1. Use `handler` when requests arrive.
2. Listen on port `8080`.

```clojure
(defn handler [request] ...)
```

This function does not start the server.

It only receives a request map and returns a response map.

### Predict before running

1. Which function starts the server?
2. Which function handles each request?
3. Which line chooses port `8080`?
4. What do you think will appear in the terminal when a request arrives?
5. What do you think the browser or `curl` will receive?

---

## 5. Run the server

```bash
lein run
```

You should see:

```text
Starting server on http://localhost:8080
```

The terminal may look stuck. That is normal.

The server is running and waiting for requests.

Do not close this terminal yet.

---

## 6. Send real HTTP requests

Open a second terminal.

Try `curl` first:

```bash
curl -i http://localhost:8080/
```

`-i` means “show the response headers too.”

You should see something like:

```text
HTTP/1.1 200 OK
Content-Type: text/plain

Hello from Jetty + Ring.
You requested: get /
```

Now try a different URI:

```bash
curl -i 'http://localhost:8080/about?name=student'
```

You can also use HTTPie:

```bash
http 'http://localhost:8080/about?name=student'
```

HTTPie also allows this query-param style:

```bash
http :8080/about name==student
```

Or open this in a browser:

```text
http://localhost:8080/about?name=student
```

### Observe both terminals

In the client terminal, you see the HTTP response.

In the server terminal, you see the `println` output from the handler.

That proves this:

```text
real HTTP request  ->  Jetty  ->  Ring request map  ->  handler
```

### Check your understanding

1. Which command sent an HTTP request?
2. Which terminal showed the HTTP response?
3. Which terminal showed `REQUEST METHOD`, `URI`, and `QUERY STRING`?
4. Why did the handler run when you used `curl`?
5. Did you manually create the request map this time?

---

## 7. Compare Lesson 1A and Lesson 1B

| Question | Lesson 1A | Lesson 1B |
|---|---|---|
| Who created the request map? | You did | Ring Jetty adapter did |
| Was Jetty running? | No | Yes |
| Was there real HTTP traffic? | No | Yes |
| Was the handler still a function? | Yes | Yes |
| Did the handler return a response map? | Yes | Yes |

### Explain in your own words

1. What stayed the same between the two lessons?
2. What changed between the two lessons?
3. Why was Lesson 1A useful before starting Jetty?

---

## 8. Add simple manual routes

Right now, every URI gets the same status code.

Replace only the `handler` function with this version:

```clojure
(defn handler [request]
  (println "REQUEST METHOD:" (:request-method request))
  (println "URI:" (:uri request))
  (println "QUERY STRING:" (:query-string request))
  (println)
  (case (:uri request)
    "/"
    {:status 200
     :headers {"Content-Type" "text/plain"}
     :body "Home page\n"}

    "/about"
    {:status 200
     :headers {"Content-Type" "text/plain"}
     :body "About page\n"}

    {:status 404
     :headers {"Content-Type" "text/plain"}
     :body "Not found\n"}))
```

Stop the server with `Ctrl-C`.

Start it again:

```bash
lein run
```

Test these:

```bash
curl -i http://localhost:8080/
curl -i http://localhost:8080/about
curl -i http://localhost:8080/no-such-page
```

### Check your understanding

1. Which request should return `Home page`?
2. Which request should return `About page`?
3. Which request should return `404 Not Found`?
4. Which part of the request map are we using to decide the response?
5. Why is this called “manual routing”?

---

## 9. Stopping the server

Usually:

```text
Ctrl-C
```

If you accidentally leave a server running, starting another server on the same port may cause an error like:

```text
Address already in use
```

That means another process is already listening on port `8080`.

Find it:

```bash
sudo lsof -i :8080
```

Then stop it politely first:

```bash
kill <PID>
```

Use this only if the process refuses to stop:

```bash
kill -9 <PID>
```

Do not make `kill -9` your first habit. It is a force kill.

### Check your understanding

1. What does “Address already in use” mean?
2. What command shows which process is using port `8080`?
3. Why should you try plain `kill` before `kill -9`?

---

## 10. Common mistakes to catch early

### Mistake 1 — Misspelling `:require`

Wrong:

```clojure
(:requre [ring.adapter.jetty :as jetty])
```

Correct:

```clojure
(:require [ring.adapter.jetty :as jetty])
```

### Mistake 2 — Putting `:require` outside the `ns` form

Wrong:

```clojure
(ns jetty-demo.core)
(:require [ring.adapter.jetty :as jetty])
(:gen-class)
```

Correct:

```clojure
(ns jetty-demo.core
  (:require [ring.adapter.jetty :as jetty])
  (:gen-class))
```

### Mistake 3 — Using `:header` instead of `:headers`

Wrong:

```clojure
{:status 200
 :header {"Content-Type" "text/plain"}
 :body "Hello"}
```

Correct:

```clojure
{:status 200
 :headers {"Content-Type" "text/plain"}
 :body "Hello"}
```

### Mistake 4 — Forgetting that `lein run` blocks

When the server is running, the terminal waits.

That is not a crash.

That is the server listening for requests.

---

## 11. No-copy understanding tasks

Do these without looking at the code.

### Task A — Explain each piece

Explain these in your own words:

```clojure
(defn handler [request] ...)
```

```clojure
(jetty/run-jetty handler {:port 8080})
```

```clojure
{:status 200
 :headers {"Content-Type" "text/plain"}
 :body "Home page\n"}
```

### Task B — Add one route

Add this route manually:

```text
/contact
```

It should return:

```text
Contact page
```

Test it:

```bash
curl -i http://localhost:8080/contact
```

### Task C — Break it on purpose

Change `:headers` to `:header`.

Run the server and request the page.

Then answer:

1. Did the program still run?
2. Did the response still have the content type you expected?
3. Why does this mistake matter?

Change it back after the test.

---

## Exit ticket

Answer these before moving on:

1. What does Jetty do?
2. What does the Ring Jetty adapter do?
3. What does the handler do?
4. What is the difference between a request map and a response map?
5. What does `run-jetty` need as its two main arguments?
6. Why does `curl -i` help you understand the response better than plain `curl`?
7. What mistake causes “Address already in use”?
8. Why is this not just copy-paste work?

---

## References

- Ring Jetty adapter documentation: https://ring-clojure.github.io/ring/ring.adapter.jetty.html
- Ring Jetty adapter dependency page: https://clojars.org/ring/ring-jetty-adapter
