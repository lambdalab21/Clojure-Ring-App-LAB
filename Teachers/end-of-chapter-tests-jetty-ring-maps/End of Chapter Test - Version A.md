# End-of-Chapter Test — Version A
## Chapter: Jetty, Ring Maps, and Handler Simulation

### Instructions for the student

Do not look at the lesson files.

This is not a copy/paste test. There is no server to run.

Your job is to prove that you understand the path:

```text
HTTP request → Jetty → Ring adapter → request map → handler → response map → HTTP response
```

Write short, clear answers.

---

## Part 1 — Big picture

### 1. Match each thing to its job

Write the correct letter.

| Term | Answer |
|---|---|
| Jetty | |
| Ring Jetty adapter | |
| Ring handler | |
| Ring request map | |
| Ring response map | |

Choices:

A. Clojure data that describes what the client requested  
B. The Java web server that listens on a port and parses HTTP  
C. Clojure data that describes what should be sent back  
D. The bridge that converts between Jetty/Java objects and Ring/Clojure maps  
E. A function that receives a request map and returns a response map

---

### 2. Fill in the flow

Fill in the missing parts.

```text
curl/browser
↓
__________
↓
Jetty parses __________
↓
Ring Jetty adapter creates __________
↓
__________ receives the request map
↓
handler returns __________
↓
Ring Jetty adapter and Jetty create the final __________
```

---

### 3. Short answer

Why is this sentence wrong?

> My handler reads directly from the socket.

Answer:

---

## Part 2 — HTTP request to Ring request map

For each URL, write the expected `:uri` and `:query-string`.

| URL | `:uri` | `:query-string` |
|---|---|---|
| `http://localhost:3000/` | | |
| `http://localhost:3000/about` | | |
| `http://localhost:3000/search?q=clojure` | | |
| `http://localhost:3000/todos?id=3&done=false` | | |
| `http://localhost:3000/users/42` | | |

---

## Part 3 — Ring response map to HTTP meaning

Given this response map:

```clojure
{:status 404
 :headers {"Content-Type" "text/plain"}
 :body "Not found"}
```

Answer:

1. What HTTP status code will the client receive?
2. What response header is included?
3. What response body is included?
4. Did Jetty have to choose this 404, or could the handler choose it?

---

## Part 4 — Handler simulation

Use this handler:

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

### Request A

```clojure
{:request-method :get
 :uri "/about"
 :query-string nil
 :headers {"host" "localhost:3000"}}
```

Answer:

1. What is `(:uri request)`?
2. Which branch runs?
3. What status comes back?
4. What body comes back?

---

### Request B

```clojure
{:request-method :get
 :uri "/missing"
 :query-string nil
 :headers {}}
```

Answer:

1. What is `(:uri request)`?
2. Which branch runs?
3. What status comes back?
4. Who chose this response in this example?

---

## Part 5 — Explain like Feynman

Explain this in 5-7 sentences using these words:

- Jetty
- Ring adapter
- request map
- handler
- response map
- HTTP response

Do not use the word “magic.”

---

## Part 6 — Final judgment

Answer honestly.

1. Which idea is still weak for you?
2. Which mistake are you most likely to make?
3. What should you review before coding?
