# Lesson 1B — Jetty Server: Now the Order Ticket Comes From the Internet

## Simple idea first

In Lesson 1A, you made a pretend request map by hand.

That was like writing your own fake order ticket and handing it to the worker.

Now we add Jetty.

Imagine a restaurant with a front door.

1. A customer walks in.
2. The front desk checks that the customer has a real order.
3. A translator turns the order into the format the kitchen understands.
4. The worker reads the order ticket.
5. The worker gives back a tray.
6. The front desk gives the tray back to the customer.

In Ring + Jetty:

```text
client       -> Jetty        -> Ring adapter       -> handler -> response map
customer     -> front desk   -> translator         -> worker  -> tray
curl/browser -> web server   -> request-map maker  -> function -> response map
```

The handler idea did not change.

Only the source of the request map changed.

---

## Goal

By the end, you should be able to explain this:

```text
curl or browser
   ↓
Jetty receives real HTTP traffic
   ↓
Ring Jetty adapter creates a request map
   ↓
handler receives that request map
   ↓
handler returns a response map
   ↓
Jetty sends an HTTP response back
```

You should also be able to point to the exact line that starts the server.

---

## 1. Continue from Lesson 1A

Use the same project:

```bash
cd jetty-demo
```

If Lesson 1A is not clear yet, stop.

Do not start Jetty until this sentence makes sense:

> A handler is a function that receives a request map and returns a response map.

---

## 2. Add the Jetty adapter dependency

Open:

```text
project.clj
```

Find `:dependencies`.

Add this line:

```clojure
[ring/ring-jetty-adapter "1.15.4"]
```

Example:

```clojure
:dependencies [[org.clojure/clojure "1.11.1"]
               [ring/ring-jetty-adapter "1.15.4"]]
```

Your Clojure version may be different. That is fine.

The important added dependency is:

```clojure
[ring/ring-jetty-adapter "1.15.4"]
```

### Simple explanation

Your project does not automatically know how to start Jetty.

The dependency is like saying:

> Please bring in the Ring tool that knows how to run Jetty.

### Real vocabulary

The dependency gives you access to the namespace:

```clojure
ring.adapter.jetty
```

That namespace contains:

```clojure
run-jetty
```

### Check your understanding

1. Which file did you edit?
2. What dependency did you add?
3. What tool does that dependency give you?
4. Why are we using `ring/ring-jetty-adapter` instead of learning all of Ring at once?

---

## 3. Download dependencies

Run:

```bash
lein deps
```

Leiningen downloads libraries into:

```text
~/.m2/repository
```

Do not edit files there.

That directory is the storage shelf for downloaded libraries.

### Check your understanding

1. What command downloaded the dependency?
2. Where did Leiningen store it?
3. Should you manually change files under `~/.m2/repository`?
4. What is the job of a dependency manager?

---

## 4. Replace `src/jetty_demo/core.clj`

Use this code:

```clojure
(ns jetty-demo.core
  (:require [ring.adapter.jetty :as jetty])
  (:gen-class))

(defn handler [request]
  (println "The handler received a request map.")
  (println "Method:" (:request-method request))
  (println "URI:" (:uri request))
  (println "Query string:" (:query-string request))
  (println)
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "Hello from Ring and Jetty.\n"
              "Method: " (name (:request-method request)) "\n"
              "URI: " (:uri request) "\n")})

(defn -main
  [& args]
  (println "Starting server on http://localhost:8080")
  (jetty/run-jetty handler {:port 8080}))
```

Do not run it yet.

Read it first.

---

## 5. Explain each important line simply

### Namespace and dependency use

```clojure
(:require [ring.adapter.jetty :as jetty])
```

Simple explanation:

> Bring in the Jetty adapter tools and call them `jetty` in this file.

Real vocabulary:

> This requires the `ring.adapter.jetty` namespace and gives it the alias `jetty`.

---

### Handler

```clojure
(defn handler [request]
  ...)
```

Simple explanation:

> This is the worker. It receives the order ticket.

Real vocabulary:

> This is the Ring handler. It receives a request map.

---

### Response map

```clojure
{:status 200
 :headers {"Content-Type" "text/plain"}
 :body "..."}
```

Simple explanation:

> This is the tray that the worker sends back.

Real vocabulary:

> This is the Ring response map.

---

### Starting Jetty

```clojure
(jetty/run-jetty handler {:port 8080})
```

Simple explanation:

> Start the front desk. Tell it which worker to use and which door number to listen on.

Real vocabulary:

> Start an embedded Jetty server, use `handler` for requests, and listen on port `8080`.

This line has two main arguments:

```clojure
handler
{:port 8080}
```

| Argument | Meaning |
|---|---|
| `handler` | Function to call when a request arrives |
| `{:port 8080}` | Server option map saying which port to use |

---

## 6. Predict before running

Write answers first.

1. Which line starts the server?
2. Which function handles each request?
3. Which port will Jetty listen on?
4. What will appear in the server terminal when a request comes in?
5. What will the client receive?
6. Who creates the request map this time: you, Jetty alone, or the Ring Jetty adapter?

---

## 7. Start the server

Run:

```bash
lein run
```

You should see:

```text
Starting server on http://localhost:8080
```

The terminal may look stuck.

It is not stuck.

It is waiting for requests.

Leave this terminal open.

---

## 8. Send a real HTTP request

Open a second terminal.

Run:

```bash
curl -i http://localhost:8080/
```

`-i` means:

> Show me the response headers too, not only the body.

You should see something like:

```text
HTTP/1.1 200 OK
Content-Type: text/plain

Hello from Ring and Jetty.
Method: get
URI: /
```

Now try:

```bash
curl -i 'http://localhost:8080/about?name=student'
```

Or use HTTPie:

```bash
http 'http://localhost:8080/about?name=student'
```

You can also use the browser:

```text
http://localhost:8080/about?name=student
```

---

## 9. Watch both terminals

There are two places to observe.

| Place | What you see |
|---|---|
| Client terminal or browser | The HTTP response |
| Server terminal | The `println` output from inside `handler` |

When you run:

```bash
curl -i 'http://localhost:8080/about?name=student'
```

the server terminal should print something like:

```text
The handler received a request map.
Method: :get
URI: /about
Query string: name=student
```

This proves the handler ran.

It also proves the handler received a request map made from the real HTTP request.

### Check your understanding

1. Which terminal sent the request?
2. Which terminal showed the response?
3. Which terminal showed the handler's `println` output?
4. What URI did the handler receive for `/about?name=student`?
5. What query string did the handler receive?
6. Did the handler receive the full URL as `:uri`, or only the path part?

---

## 10. Compare Lesson 1A and Lesson 1B

| Question | Lesson 1A | Lesson 1B |
|---|---|---|
| Was Jetty running? | No | Yes |
| Was there real HTTP traffic? | No | Yes |
| Who created the request map? | You did | Ring Jetty adapter did |
| Was the handler still a function? | Yes | Yes |
| Did the handler still return a response map? | Yes | Yes |

### Explain it to a 10-year-old

Use the order-ticket story.

1. What was the order ticket in Lesson 1A?
2. Who made the order ticket in Lesson 1A?
3. What is the order ticket in Lesson 1B?
4. Who makes the order ticket in Lesson 1B?
5. What stayed the same?

Expected idea:

> The handler is still the worker. The request map is still the order ticket. The difference is that Jetty and the Ring adapter now create the ticket from a real HTTP request.

---

## 11. Add simple manual routing

Right now, every URI returns a similar response.

Now make the handler choose different responses based on the URI.

This is called routing.

Simple explanation:

> Routing means reading the address on the order ticket and deciding which answer to send back.

Replace only the `handler` function with this:

```clojure
(defn handler [request]
  (println "The handler received a request map.")
  (println "Method:" (:request-method request))
  (println "URI:" (:uri request))
  (println "Query string:" (:query-string request))
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

Yes, the response maps are repetitive.

That is okay for this lesson.

Do not hide the important idea too early.

The important idea is:

```clojure
(case (:uri request)
  ...)
```

That line asks:

> What path did the client request?

Stop the server with:

```text
Ctrl-C
```

Start it again:

```bash
lein run
```

Then test:

```bash
curl -i http://localhost:8080/
curl -i http://localhost:8080/about
curl -i http://localhost:8080/no-such-page
```

### Check your understanding

1. Which request returns `Home page`?
2. Which request returns `About page`?
3. Which request returns `Not found`?
4. Which part of the request map controls the routing decision?
5. Why does `/about?name=student` still match `/about`?
6. What status code should an unknown page return?

---

## 12. No-copy task: add one route

Add this route by yourself:

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

### Questions

1. Where did you add the new route?
2. Did you change `run-jetty`?
3. Did you change the dependency?
4. Why was only the handler changed?

Expected idea:

> Routing is application logic. It belongs inside the handler or inside code called by the handler. Starting Jetty did not need to change.

---

## 13. Break it on purpose: `:headers` vs `:header`

Change one response map from this:

```clojure
:headers {"Content-Type" "text/plain"}
```

to this wrong version:

```clojure
:header {"Content-Type" "text/plain"}
```

Restart and test with:

```bash
curl -i http://localhost:8080/
```

Then change it back.

### Questions

1. Did the program crash?
2. Did the response still contain the header you expected?
3. Why is `:headers` the correct key?
4. Why is this mistake easy to miss if you only look at the body?
5. Why does `curl -i` help catch this mistake?

Important point:

> A response map must use the keys Ring expects. `:headers` is one of those keys. `:header` is just a wrong key that Ring does not use as the response headers.

---

## 14. Stopping the server

Usually, stop the server with:

```text
Ctrl-C
```

If you try to start another server and see:

```text
Address already in use
```

simple explanation:

> Someone is already standing in front of door 8080.

Real explanation:

> Another process is already listening on port `8080`.

Find it:

```bash
sudo lsof -i :8080
```

Stop it politely first:

```bash
kill <PID>
```

Use this only if the process refuses to stop:

```bash
kill -9 <PID>
```

Do not make `kill -9` your first habit.

It is a force kill.

### Check your understanding

1. What does “Address already in use” mean?
2. What command shows which process is using port `8080`?
3. Why should you try plain `kill` before `kill -9`?
4. What bad habit are you trying to avoid?

---

## 15. Common mistakes

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

### Mistake 3 — Confusing `:headers` and `:header`

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

### Mistake 4 — Thinking `lein run` froze

When the server is running, the terminal waits.

That is the server listening.

That is not a freeze.

---

## 16. Final teach-back

Explain this flow without looking:

```text
curl -> Jetty -> Ring adapter -> handler -> response map -> HTTP response
```

Your explanation must include these words:

- client
- server
- port
- adapter
- request map
- handler
- response map

A weak answer:

> Jetty runs the website.

That is too vague.

A better answer:

> `curl` is the client. It sends an HTTP request to port 8080. Jetty is the server listening on that port. The Ring Jetty adapter turns the request into a Clojure request map. The handler receives that map and returns a response map. The adapter and Jetty send that response back as HTTP.

---

## Exit ticket

Answer these before moving on.

1. What does Jetty do?
2. What does the Ring Jetty adapter do?
3. What does the handler do?
4. What is the difference between a request map and a response map?
5. What two arguments does `run-jetty` receive in this lesson?
6. Why does `curl -i` teach more than plain `curl`?
7. What causes “Address already in use”?
8. Why did adding `/contact` require changing the handler but not `run-jetty`?
9. Why is this lesson not just copy-paste work?

---

## References

- Ring Jetty adapter docs: https://ring-clojure.github.io/ring/ring.adapter.jetty.html
- Ring Jetty adapter dependency page: https://clojars.org/ring/ring-jetty-adapter
