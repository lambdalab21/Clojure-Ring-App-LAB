# Project 5 — Status Code Practice Server

## Goal

Build a server that deliberately returns different HTTP status codes.

This project strengthens this idea:

> The handler can choose the response status.

---

## What you will build

| URI | Status | Body |
|---|---:|---|
| `/ok` | 200 | `OK` |
| `/created` | 201 | `Created` |
| `/bad-request` | 400 | `Bad request` |
| `/missing` | 404 | `Not found` |
| anything else | 404 | `Unknown page` |

---

## Starter helper

```clojure
(defn text-response [status body]
  {:status status
   :headers {"Content-Type" "text/plain"}
   :body (str body "\n")})

(defn handler [request]
  (case (:uri request)
    "/ok" (text-response 200 "OK")

    ;; add more cases here

    (text-response 404 "Unknown page")))
```

This introduces a helper function to remove repetition.

Do not use a library.

---

## Required tests

Use `curl -i` so you can see the status.

```bash
curl -i http://localhost:8080/ok
curl -i http://localhost:8080/created
curl -i http://localhost:8080/bad-request
curl -i http://localhost:8080/missing
curl -i http://localhost:8080/whatever
```

---

## Questions

1. Which response-map key controls the status code?
2. Why must you use `curl -i` for this project?
3. Who chose status `201`?
4. Who chose status `404`?
5. What does `text-response` return?

---

## Stretch

Add this route:

```text
/teapot
```

Return status:

```text
418
```

Body:

```text
I am a teapot
```

---

## Finish line

Explain this:

> A Ring response map is not only body text. It also controls status and headers.
