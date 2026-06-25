# Project 2 — Three-Page Manual Router

## Goal

Build a tiny website with three routes using only `case`.

This project strengthens this idea:

> Routing is just handler logic that looks at the request map.

---

## What you will build

These URLs should work:

| URL | Response body | Status |
|---|---|---|
| `/` | `Home page` | `200` |
| `/about` | `About page` | `200` |
| `/contact` | `Contact page` | `200` |
| anything else | `Not found` | `404` |

---

## Starting handler

```clojure
(defn handler [request]
  (case (:uri request)
    "/"
    {:status 200
     :headers {"Content-Type" "text/plain"}
     :body "Home page\n"}

    ;; add more routes here

    {:status 404
     :headers {"Content-Type" "text/plain"}
     :body "Not found\n"}))
```

---

## Prediction before testing

| URL | Expected body | Expected status |
|---|---|---|
| `/` | | |
| `/about` | | |
| `/contact` | | |
| `/missing` | | |
| `/about?x=1` | | |

Important: `/about?x=1` has `:uri` of `"/about"`.

---

## Required tests

```bash
curl -i http://localhost:8080/
curl -i http://localhost:8080/about
curl -i http://localhost:8080/contact
curl -i http://localhost:8080/missing
curl -i 'http://localhost:8080/about?x=1'
```

---

## Questions

1. Which request-map key controls the route?
2. Why does `/missing` return 404?
3. Who chose that 404: Jetty or your handler?
4. Why does `/about?x=1` still match `/about`?
5. Did you need to change `run-jetty` to add a route?

---

## No-copy task

Add this fourth page:

```text
/help
```

It should return:

```text
Help page
```

Test it with:

```bash
curl -i http://localhost:8080/help
```

---

## Finish line

Explain why this is called “manual routing.”
