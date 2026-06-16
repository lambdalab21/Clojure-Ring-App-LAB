# Understanding Jetty, Ring, and Your Clojure Handler

## Purpose

This lesson is about the path an HTTP request takes before your Clojure code sees it.

Do **not** treat this as a copy/paste exercise. Your job is to understand what each layer does:

- **Jetty** handles network connections and HTTP parsing.
- The **Ring Jetty adapter** translates Jetty's Java servlet objects into Clojure maps.
- Your **Ring handler** receives a request map and returns a response map.

Final mental model:

> Jetty handles sockets and HTTP parsing; the Ring adapter translates the parsed request into a Ring request map and calls your handler.

---

## 1. The three layers

### Layer 1: Jetty

Jetty is the Java web server.

It does low-level server work:

- listens on a TCP port
- accepts client connections
- reads raw bytes from the socket
- parses HTTP request lines and headers
- handles keep-alive connections
- manages server threads
- writes the final HTTP response bytes back to the client

Your Clojure handler does **not** directly read from a socket.

### Layer 2: Ring Jetty adapter

The Ring Jetty adapter connects Jetty to Ring.

It does translation work:

- receives Jetty's Java `HttpServletRequest`
- builds a Ring request map
- calls your Clojure handler function
- takes your Ring response map
- writes the result into Jetty's Java `HttpServletResponse`

### Layer 3: Your Ring handler

A Ring handler is just a Clojure function.

It takes one argument:

```clojure
request
```

It returns one value:

```clojure
response
```

The usual shape is:

```clojure
(defn handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body "Hello from Ring"})
```

The handler does not know about sockets, TCP packets, or raw HTTP parsing. It receives Clojure data and returns Clojure data.

### Check your understanding

Answer these before continuing:

1. What does Jetty do that your Clojure handler does **not** do?
2. What does the Ring Jetty adapter translate **from** and **to**?
3. What is the argument passed into a Ring handler?
4. What does a Ring handler return?
5. Why is it wrong to say that your handler directly handles the TCP connection?

---

## 2. The full request/response path

Here is the complete flow:

```text
Browser, curl, or httpie
   ↓
TCP socket
   ↓
Jetty HTTP parser
   ↓
HttpServletRequest  (Java object)
   ↓
Ring Jetty adapter
   ↓
Ring request map    (Clojure data)
   ↓
(handler request)
   ↓
Ring response map   (Clojure data)
   ↓
Ring Jetty adapter
   ↓
HttpServletResponse (Java object)
   ↓
Jetty writes HTTP response bytes to the socket
   ↓
Browser, curl, or httpie receives the response
```

The important boundary is here:

```text
HttpServletRequest  →  Ring request map
```

That is where Java servlet data becomes ordinary Clojure data.

### Check your understanding

1. In the flow above, where does raw HTTP parsing happen?
2. Where does the Java object become a Clojure map?
3. Where does your code enter the flow?
4. Where does your code leave the flow?
5. What would happen if the request is malformed before Ring receives it?

---

## 3. What “valid HTTP request” means

Jetty only passes valid, parsed HTTP requests forward.

That means Jetty has already handled things like:

- the HTTP request line
- headers
- connection handling
- malformed HTTP syntax
- request body framing

Example HTTP request:

```http
GET /todos?id=3 HTTP/1.1
Host: localhost:3000
User-Agent: curl/8.x
Accept: */*
```

By the time your handler runs, Ring may give you a request map containing values such as:

```clojure
{:request-method :get
 :uri "/todos"
 :query-string "id=3"
 :headers {"host" "localhost:3000"
           "user-agent" "curl/8.x"
           "accept" "*/*"}}
```

This map is not the raw HTTP request. It is Ring's Clojure representation of the parsed request.

### Check your understanding

1. Is the Ring request map the same thing as the raw HTTP request bytes?
2. Which layer rejects malformed HTTP before your handler runs?
3. Why can your handler assume the request is already syntactically valid HTTP?
4. What is the difference between `:uri` and `:query-string` in the example?
5. Why are header names shown inside the `:headers` map?

---

## 4. Minimal code experiment

Create a small Ring/Jetty program.

```clojure
(ns jetty-demo.core
  (:require [ring.adapter.jetty :refer [run-jetty]]))

(defn handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "Method: " (:request-method request) "\n"
              "URI: " (:uri request) "\n"
              "Query string: " (:query-string request) "\n")})

(defonce server (atom nil))

(defn start []
  (reset! server
          (run-jetty #'handler {:port 3000
                                :join? false})))

(defn stop []
  (when @server
    (.stop @server)
    (reset! server nil)))
```

Start the server from the REPL:

```clojure
(start)
```

Then send a request:

```bash
curl 'http://localhost:3000/todos?id=3'
```

Expected output shape:

```text
Method: :get
URI: /todos
Query string: id=3
```

### Do not skip this prediction step

Before running the command, write your prediction:

1. What will `(:request-method request)` be?
2. What will `(:uri request)` be?
3. What will `(:query-string request)` be?
4. Which part of the URL becomes `:uri`?
5. Which part of the URL becomes `:query-string`?

After running it, compare your prediction with the actual result.

---

## 5. Why `#'handler` is used

This line starts Jetty:

```clojure
(run-jetty #'handler {:port 3000 :join? false})
```

The `#'handler` form passes the **Var** for `handler`, not only the current function value.

Why this matters in the REPL:

- If you redefine `handler`, the server can use the updated function.
- This makes REPL-driven development smoother.
- Without it, you may need to restart the server more often after changing the handler.

Do not memorize this blindly. The idea is:

> Passing the Var lets the running server find the latest definition of the handler.

### Check your understanding

1. What does `run-jetty` start?
2. Why do we use `:join? false` during REPL development?
3. What is the practical benefit of using `#'handler`?
4. What might happen if you pass `handler` instead of `#'handler` and then redefine the function?
5. Why is REPL-friendly development useful when learning web development?

---

## 6. Modify the handler

Change the handler so it returns the request method, URI, query string, and user-agent.

Target behavior:

```bash
curl -A 'StudentAgent' 'http://localhost:3000/search?q=clojure'
```

Expected output shape:

```text
Method: :get
URI: /search
Query string: q=clojure
User-Agent: StudentAgent
```

Hint:

```clojure
(get-in request [:headers "user-agent"])
```

### Questions to answer

1. Where is the user-agent stored in the Ring request map?
2. Why do we use `get-in` instead of `get` for this value?
3. What happens if the request does not include a user-agent header?
4. Which layer originally parsed the header?
5. Which layer placed the parsed header into the Ring request map?

---

## 7. Return different responses based on the URI

Now make your handler branch on the URI.

Example:

```clojure
(defn handler [request]
  (case (:uri request)
    "/"      {:status 200
              :headers {"Content-Type" "text/plain"}
              :body "Home page"}

    "/about" {:status 200
              :headers {"Content-Type" "text/plain"}
              :body "About page"}

    {:status 404
     :headers {"Content-Type" "text/plain"}
     :body "Not found"}))
```

Test it:

```bash
curl -i 'http://localhost:3000/'
curl -i 'http://localhost:3000/about'
curl -i 'http://localhost:3000/missing'
```

### Questions to answer

1. Which key in the request map decides which page to return?
2. Why does `/missing` return a 404 response?
3. Is Jetty deciding that `/missing` is not found, or is your handler deciding that?
4. What does `curl -i` show that plain `curl` hides?
5. Why is `:status` part of the Ring response map?

---

## 8. Handler, adapter, and server are separate ideas

Do not blur these together.

| Thing | What it is | Main job |
|---|---|---|
| Jetty | Java web server | Sockets, HTTP parsing, response writing |
| Ring Jetty adapter | Bridge between Jetty and Ring | Java servlet object ↔ Ring maps |
| Ring handler | Clojure function | Request map → response map |
| Middleware | Function wrapper | Transform request and/or response |

Middleware works well because Ring uses a simple function boundary:

```text
request map → handler → response map
```

A middleware can wrap that boundary:

```clojure
(defn wrap-debug [handler]
  (fn [request]
    (println "Request URI:" (:uri request))
    (handler request)))
```

Then:

```clojure
(def app
  (wrap-debug handler))
```

Now Jetty can run `app` instead of `handler`.

### Questions to answer

1. What does middleware receive as an argument?
2. What does middleware return?
3. In `wrap-debug`, when does the original handler get called?
4. Why is middleware possible without changing Jetty?
5. Why does Ring's design make middleware feel like normal function composition?

---

## 9. Common mistakes

### Mistake 1: “Jetty calls my route directly.”

Better:

> Jetty receives and parses the HTTP request. The Ring adapter calls the Ring handler.

### Mistake 2: “My handler parses HTTP.”

Better:

> Jetty parses HTTP. My handler receives a Ring request map.

### Mistake 3: “The request map is the HTTP request.”

Better:

> The request map is Ring's Clojure representation of the parsed HTTP request.

### Mistake 4: “The handler is magic.”

Better:

> The handler is a function: request map in, response map out.

### Mistake 5: “404 always comes from the server.”

Better:

> Sometimes the application handler chooses to return a 404 response. Jetty only sends the response back to the client.

---

## 10. Final review questions

Answer these without looking above.

1. What is Jetty responsible for?
2. What is the Ring Jetty adapter responsible for?
3. What is your handler responsible for?
4. What data type does your handler receive?
5. What data type does your handler return?
6. What are the three most important keys in a basic Ring response map?
7. Why does malformed HTTP usually not reach your handler?
8. What is the difference between `HttpServletRequest` and a Ring request map?
9. Why is `:join? false` useful when working from the REPL?
10. Why does middleware fit naturally into Ring?
11. What part of the system handles sockets?
12. What part of the system calls your Clojure function?
13. What part of the system turns the Ring response map back into an HTTP response?
14. Why should you avoid saying “Jetty is my Clojure web framework”?
15. Write the full request/response path from browser to handler and back.

---

## 11. Short written explanation

Write a 5-7 sentence explanation using these words correctly:

- Jetty
- Ring adapter
- request map
- handler
- response map
- HTTP parsing
- middleware

Your explanation must include this idea:

> The handler does not deal with raw HTTP bytes directly.

---

## 12. Exit ticket

Before you mark this lesson complete, answer these three questions from memory:

1. What does Jetty do?
2. What does the Ring adapter do?
3. What does the handler do?

If you cannot answer those clearly, you are not done.
