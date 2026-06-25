# Project 7 — Tiny Text API Without JSON Library

## Goal

Build a tiny API-like server without adding a JSON library.

This project keeps the focus on handlers and response maps.

---

## What you will build

Routes:

| URI | Body |
|---|---|
| `/api/status` | `status=ok` |
| `/api/version` | `version=1` |
| `/api/time-placeholder` | `time=not-implemented-yet` |
| anything else | `error=not-found` |

Use plain text on purpose.

No JSON library yet.

---

## Why no JSON yet?

Because the goal is not API design.

The goal is:

```text
request map → handler decision → response map
```

JSON can come later.

---

## Starter helper

```clojure
(defn text-response [status body]
  {:status status
   :headers {"Content-Type" "text/plain"}
   :body (str body "\n")})
```

---

## Required tests

```bash
curl -i http://localhost:8080/api/status
curl -i http://localhost:8080/api/version
curl -i http://localhost:8080/api/time-placeholder
curl -i http://localhost:8080/api/unknown
```

---

## Questions

1. Which request-map key controls the route?
2. Which response-map key controls the body?
3. Which route returns 404?
4. Why are we not using JSON yet?
5. Why is plain text enough for this exercise?

---

## Stretch

Add:

```text
/api/echo-uri
```

It should return:

```text
uri=/api/echo-uri
```

---

## Finish line

Explain why this is “API-like” but still beginner-friendly.
