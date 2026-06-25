# Project 3 — Method Responder

## Goal

Build a handler that responds differently to different HTTP methods.

This project strengthens this idea:

> The request map contains `:request-method`, and the handler can branch on it.

---

## What you will build

The same URI can return different messages depending on method.

| Method | Response body |
|---|---|
| GET | `GET request received` |
| POST | `POST request received` |
| PUT | `PUT request received` |
| anything else | `Unknown method` |

---

## Starter helper

Write this helper first:

```clojure
(defn method-message [request]
  (case (:request-method request)
    :get "GET request received"
    :post "POST request received"
    :put "PUT request received"
    "Unknown method"))
```

Then use it inside a handler:

```clojure
(defn handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str (method-message request) "\n")})
```

---

## Direct function test first

Before Jetty, predict and call:

```clojure
(method-message {:request-method :get})
(method-message {:request-method :post})
(method-message {:request-method :delete})
(method-message {})
```

---

## Required curl tests

```bash
curl -i http://localhost:8080/method
curl -i -X POST http://localhost:8080/method
curl -i -X PUT http://localhost:8080/method
curl -i -X DELETE http://localhost:8080/method
```

---

## Questions

1. Which key did the handler read?
2. Why did GET and POST produce different bodies?
3. Did the URI matter in this project?
4. What happened for DELETE?
5. Why is splitting `method-message` from `handler` helpful for learning?

---

## Stretch

Return status `405` for unknown methods instead of `200`.

Hint: you may need the handler itself to branch, not only the body message.

---

## Finish line

Explain this:

> A handler can branch on method, URI, or both.
