# Project 4 — Query String Detective

## Goal

Build a server that proves `:uri` and `:query-string` are separate.

This project kills a common beginner mistake:

> `:uri` is not `"/search?q=clojure"`.

---

## What you will build

For this request:

```bash
curl -i 'http://localhost:8080/search?q=clojure'
```

The body should show:

```text
Path only: /search
Query only: q=clojure
```

---

## Handler

Write a handler that returns:

```text
Path only: <uri>
Query only: <query-string>
```

Use:

```clojure
(:uri request)
(:query-string request)
```

If the query string is missing, show:

```text
Query only: none
```

Hint:

```clojure
(or (:query-string request) "none")
```

---

## Required tests

```bash
curl -i http://localhost:8080/
curl -i http://localhost:8080/search
curl -i 'http://localhost:8080/search?q=clojure'
curl -i 'http://localhost:8080/search?q=clojure&page=2'
```

---

## Prediction table

| URL | Expected `:uri` | Expected `:query-string` |
|---|---|---|
| `/` | | |
| `/search` | | |
| `/search?q=clojure` | | |
| `/search?q=clojure&page=2` | | |

---

## Questions

1. What is the `:uri` for `/search?q=clojure`?
2. What is the `:query-string`?
3. Which part comes before `?`?
4. Which part comes after `?`?
5. Why is this important for routing?

---

## Stretch

Make `/search` return `Search page`, but make `/other?q=clojure` return `Not found`.

This proves that routing should use `:uri`, not the whole URL string.

---

## Finish line

Explain the rule:

> `:uri` is ________. `:query-string` is ________.
