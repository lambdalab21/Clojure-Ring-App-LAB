# Understanding Jetty, Ring, and Your Handler — Feynman Student Lesson

## Purpose

This lesson explains what happens **before your Clojure function runs**.

Do not memorize the words first. First understand the story.

Simple version:

> A browser sends an order. Jetty receives and understands the order. The Ring adapter rewrites the order into a Clojure map. Your handler reads the map and returns another map. The adapter and Jetty turn that map back into an HTTP response.

Technical version:

> Jetty handles sockets and HTTP parsing. The Ring Jetty adapter turns Jetty's Java servlet request into a Ring request map. Your Ring handler receives that map and returns a Ring response map.

If you cannot explain that without looking, you are not done.

---

## 0. The 10-year-old picture

Imagine a small restaurant.

| Restaurant story | Web server idea |
|---|---|
| Customer | Browser, `curl`, or HTTPie |
| Customer's order | HTTP request |
| Front desk worker | Jetty |
| Translator who rewrites the order clearly | Ring Jetty adapter |
| Cook | Your Ring handler |
| Finished meal | HTTP response |
| Tray with food and label | Ring response map |

The cook does **not** talk to every customer directly.

The cook receives a clean order ticket.

That is your handler's job:

```text
clean order ticket in  →  cook decides what to make  →  tray comes out
request map in         →  handler runs               →  response map out
```

### Stop and explain

Explain this aloud in your own words:

1. Who is the customer?
2. Who is the front desk worker?
3. Who rewrites the order into a form the cook can use?
4. Who is the cook?
5. What is the clean order ticket?
6. What is the finished tray?

If you cannot answer these, do not continue.

---

## 1. The real names

Now replace the restaurant names with real web-development names.

### Jetty

Jetty is the Java web server.

Jetty does the low-level server work:

- listens on a port
- accepts network connections
- reads raw bytes from the socket
- parses HTTP
- manages connection details
- writes the final response bytes back to the client

Simple version:

> Jetty is the front desk. It talks to the outside world.

Important point:

> Your Clojure handler does **not** read raw TCP sockets.

---

### Ring Jetty adapter

The Ring Jetty adapter is the bridge between Jetty and Ring.

It translates:

```text
Jetty Java object  →  Ring Clojure map
Ring Clojure map   →  Jetty Java response object
```

Simple version:

> The adapter is the translator. It rewrites Jetty's Java request into a Clojure map.

---

### Ring handler

A Ring handler is just a Clojure function.

It receives one value:

```clojure
request
```

It returns one value:

```clojure
response
```

Example shape:

```clojure
(defn handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body "Hello from Ring"})
```

Simple version:

> The handler is the cook. It receives a clean request map and returns a response map.

### Check your understanding

Answer without copying from above:

1. What does Jetty do?
2. What does the Ring Jetty adapter do?
3. What does the handler do?
4. Why is it wrong to say, “my handler reads from the socket”?
5. What are the two maps in Ring's basic request/response model?

---

## 2. The full path of one request

When you type this:

```bash
curl 'http://localhost:3000/todos?id=3'
```

A simplified path looks like this:

```text
curl
  ↓
network connection
  ↓
Jetty receives bytes
  ↓
Jetty parses HTTP
  ↓
Jetty creates Java servlet request object
  ↓
Ring Jetty adapter creates Ring request map
  ↓
handler receives request map
  ↓
handler returns response map
  ↓
Ring Jetty adapter turns response map into Java response object
  ↓
Jetty writes HTTP response bytes
  ↓
curl prints the result
```

The key boundary is this:

```text
Java servlet world  →  Ring Clojure data world
```

That boundary is the adapter's job.

### Explain it like you are teaching a younger student

Use these words:

- Jetty
- adapter
- request map
- handler
- response map

Write 3-5 sentences.

Do not use the word “magic.”

---

## 3. Raw HTTP versus Ring request map

A real HTTP request may look like this:

```http
GET /todos?id=3 HTTP/1.1
Host: localhost:3000
User-Agent: curl/8.x
Accept: */*
```

Your handler does **not** receive that raw text.

Your handler receives a Clojure map that may contain data like this:

```clojure
{:request-method :get
 :uri "/todos"
 :query-string "id=3"
 :headers {"host" "localhost:3000"
           "user-agent" "curl/8.x"
           "accept" "*/*"}}
```

Simple version:

> Raw HTTP is the messy original order. The Ring request map is the clean order ticket.

### Important detail: URI and query string are separate

For this URL:

```text
http://localhost:3000/todos?id=3
```

Ring usually gives you:

```clojure
:uri "/todos"
:query-string "id=3"
```

Do not say `:uri` is `"/todos?id=3"`.

That is a common beginner mistake.

### Prediction quiz

For each URL, predict `:uri` and `:query-string`.

| URL | `:uri` | `:query-string` |
|---|---|---|
| `http://localhost:3000/` | | |
| `http://localhost:3000/about` | | |
| `http://localhost:3000/search?q=clojure` | | |
| `http://localhost:3000/todos?id=3&done=false` | | |

---

## 4. What “valid HTTP request” means

Jetty checks the HTTP request before Ring sees it.

Jetty handles things like:

- request line syntax
- headers
- body framing
- connection rules
- malformed HTTP

If the request is badly formed, Jetty may reject it before your handler runs.

Simple version:

> The front desk rejects impossible orders before the cook sees them.

### Check your understanding

1. Who checks whether the HTTP syntax is valid?
2. Does your handler usually receive raw malformed HTTP text?
3. Why can the handler focus on a Clojure map instead of socket details?
4. If the handler never runs, what layer might you inspect first?
5. If the handler runs but returns the wrong body, what layer might you inspect first?

---

## 5. The handler is not the whole server

This is wrong:

```text
My handler is the web server.
```

Better:

```text
Jetty is the web server.
The Ring adapter connects Jetty to Ring.
My handler is the Clojure function that decides the response.
```

This is also wrong:

```text
Jetty is my Clojure web framework.
```

Better:

```text
Jetty is the Java server. Ring gives Clojure a simple handler model.
```

### One-sentence drill

Write one sentence for each word:

1. Jetty
2. Ring Jetty adapter
3. Ring handler
4. request map
5. response map

Each sentence must be simple enough for a 10-year-old to understand.

---

## 6. Paper simulation: no server yet

Before using Jetty, practice the idea on paper.

Here is a fake request map:

```clojure
{:request-method :get
 :uri "/about"
 :query-string nil
 :headers {"host" "localhost:3000"}}
```

Pretend this goes into a handler:

```clojure
(defn handler [request]
  (case (:uri request)
    "/"      {:status 200
              :headers {"Content-Type" "text/plain"}
              :body "Home"}

    "/about" {:status 200
              :headers {"Content-Type" "text/plain"}
              :body "About"}

    {:status 404
     :headers {"Content-Type" "text/plain"}
     :body "Not found"}))
```

### Answer before running anything

1. What is `(:uri request)`?
2. Which branch of `case` runs?
3. What `:status` comes back?
4. What `:body` comes back?
5. Did Jetty choose this response, or did the handler choose it?

Now try this fake request map:

```clojure
{:request-method :get
 :uri "/missing"
 :query-string nil
 :headers {}}
```

Answer:

1. What status comes back?
2. What body comes back?
3. Who chose the 404 in this example?

---

## 7. Why middleware works

Middleware sounds advanced, but the simple idea is not hard.

A handler is:

```text
request map → response map
```

Middleware wraps a handler and returns another handler.

```clojure
(defn wrap-debug [handler]
  (fn [request]
    (println "Request URI:" (:uri request))
    (handler request)))
```

Simple version:

> Middleware is like a helper standing next to the cook. The helper can read the order, write notes, and then hand the order to the cook.

This works because Ring uses plain functions.

```text
request map → middleware wrapper → original handler → response map
```

### Check your understanding

1. What does `wrap-debug` receive?
2. What does `wrap-debug` return?
3. Does Jetty need to know middleware exists?
4. Why can Jetty run the wrapped app like a normal handler?
5. In the example, who prints the URI?
6. Who returns the final response?

---

## 8. Common wrong ideas to kill early

### Wrong idea 1

> “Jetty calls my route directly.”

Better:

> Jetty receives and parses the HTTP request. The Ring adapter calls the Ring handler.

---

### Wrong idea 2

> “My handler parses HTTP.”

Better:

> Jetty parses HTTP. My handler receives a Ring request map.

---

### Wrong idea 3

> “The request map is the raw HTTP request.”

Better:

> The request map is Ring's Clojure representation of the parsed HTTP request.

---

### Wrong idea 4

> “404 always comes from Jetty.”

Better:

> Sometimes the handler chooses to return a 404 response. Jetty only sends it back.

---

### Wrong idea 5

> “Middleware is special server magic.”

Better:

> Middleware is function wrapping: handler in, handler out.

---

## 9. Final Feynman explanation

Close the file.

Then explain this from memory in 5-7 sentences:

1. What happens when `curl` sends a request to Jetty?
2. What does the Ring Jetty adapter do?
3. What does the handler receive?
4. What does the handler return?
5. Why does middleware fit naturally into Ring?

You must use these words correctly:

- Jetty
- HTTP parsing
- Ring adapter
- request map
- handler
- response map
- middleware

---

## 10. Exit ticket

Before moving to the next lesson, answer these from memory:

1. Jetty handles __________ and __________.
2. The Ring adapter changes Jetty's Java request into a __________.
3. A handler is a Clojure __________.
4. A handler receives a __________ and returns a __________.
5. Middleware works because handlers are ordinary __________.

If you miss more than one, repeat the lesson.

---

## 11. Teacher checkpoint

The student may move on only when he can say something close to this without reading:

> Jetty talks to the network and parses HTTP. The Ring Jetty adapter converts Jetty's Java servlet request into a Clojure request map. My handler is just a function that receives that request map and returns a response map. The adapter and Jetty turn the response map back into an HTTP response. Middleware works because it wraps one handler and returns another handler.
