# End-of-Chapter Test — Version B
## Chapter: Jetty, Ring Maps, and Handler Simulation

### Instructions for the student

This version is more scenario-based.

Do not run code. Think.

---

## Part 1 — Restaurant analogy

Complete the table.

| Restaurant story | Web development idea |
|---|---|
| Customer | |
| Messy order | |
| Front desk worker | |
| Translator | |
| Clean order ticket | |
| Cook | |
| Finished tray | |

---

## Part 2 — Correct the wrong statements

Rewrite each wrong statement correctly.

### 1.

Wrong:

> The Ring handler is the whole web server.

Better:

---

### 2.

Wrong:

> The Ring request map is raw HTTP text.

Better:

---

### 3.

Wrong:

> `:uri` includes everything after the domain name, including the query string.

Better:

---

### 4.

Wrong:

> A 404 always means Jetty decided the page was missing.

Better:

---

### 5.

Wrong:

> The response map is already the final HTTP response.

Better:

---

## Part 3 — Request-map prediction

A client sends this request:

```http
GET /books/7?format=json HTTP/1.1
Host: localhost:3000
User-Agent: TestClient
Accept: */*
```

Fill in the likely Ring request-map values.

```clojure
{:request-method ______
 :uri ______
 :query-string ______
 :headers {"host" ______
           "user-agent" ______
           "accept" ______}}
```

---

## Part 4 — Response-map explanation

Given:

```clojure
{:status 200
 :headers {"Content-Type" "application/json"}
 :body "{\"ok\":true}"}
```

Answer:

1. What status code is sent?
2. What does `Content-Type` claim?
3. Is the body a Clojure map or a string?
4. Why might the client care about `Content-Type`?

---

## Part 5 — Handler trace

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

Trace this request:

```clojure
{:request-method :get
 :uri "/"
 :query-string "debug=true"
 :headers {"user-agent" "curl"}}
```

Answer:

1. What key does the handler inspect?
2. What value does it find?
3. Does this handler care about `:query-string`?
4. What response body comes back?
5. What would need to change if `debug=true` should matter?

---

## Part 6 — Debugging judgment

Choose the better first place to inspect: Jetty/request path or handler logic.

1. The handler never runs at all.
2. The handler runs, but the body text is wrong.
3. The URL `/about?x=1` returns body `"About"`, but the student expected URI to be `"/about?x=1"`.
4. A malformed HTTP request is rejected before Clojure code runs.
5. A request to `/missing` returns a 404 created by the handler.

---

## Part 7 — Teach-back

Explain this to a younger student:

> Outside the program, HTTP travels. Inside the Ring handler, Clojure maps are used.

Use 4-6 sentences.
