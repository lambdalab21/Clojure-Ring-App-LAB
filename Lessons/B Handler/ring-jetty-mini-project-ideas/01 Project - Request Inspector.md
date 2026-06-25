# Project 1 — Request Inspector

## Goal

Build a server that shows important parts of the Ring request map.

This project strengthens this idea:

> The handler receives a request map, not raw HTTP text.

---

## What you will build

When you run:

```bash
curl -i 'http://localhost:8080/search?q=clojure'
```

The response body should show something like:

```text
Method: get
URI: /search
Query string: q=clojure
User-Agent: curl/...
```

The exact user-agent may differ.

---

## Starting handler

Replace only the handler with this starter:

```clojure
(defn handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body "TODO"})
```

Now make the body include:

- request method
- URI
- query string
- user-agent header

Useful expressions:

```clojure
(:request-method request)
(:uri request)
(:query-string request)
(get-in request [:headers "user-agent"])
```

---

## Prediction before testing

Before running each command, fill in the table.

| Command | Expected URI | Expected query string |
|---|---|---|
| `curl -i http://localhost:8080/` | | |
| `curl -i http://localhost:8080/about` | | |
| `curl -i 'http://localhost:8080/search?q=clojure'` | | |
| `curl -i 'http://localhost:8080/todos?id=3&done=false'` | | |

---

## Required tests

```bash
curl -i http://localhost:8080/
curl -i http://localhost:8080/about
curl -i 'http://localhost:8080/search?q=clojure'
curl -i 'http://localhost:8080/todos?id=3&done=false'
curl -i -A 'StudentAgent' http://localhost:8080/agent
```

---

## Questions

1. Which request-map key contains the path?
2. Which request-map key contains the part after `?`?
3. Where is the user-agent stored?
4. Did your handler parse raw HTTP?
5. What did Jetty and the Ring adapter do before your handler ran?

---

## Stretch

Add the host header to the output.

Hint:

```clojure
(get-in request [:headers "host"])
```

---

## Finish line

Explain this project in 4 sentences:

1. What the client sends.
2. What Jetty and the adapter create.
3. What your handler reads.
4. What your handler returns.
