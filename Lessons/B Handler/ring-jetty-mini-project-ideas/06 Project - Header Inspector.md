# Project 6 — Header Inspector

## Goal

Build a server that reads request headers.

This project strengthens this idea:

> Request headers live inside the nested `:headers` map.

---

## What you will build

A server that shows:

- host
- user-agent
- accept header

---

## Handler

Use expressions like:

```clojure
(get-in request [:headers "host"])
(get-in request [:headers "user-agent"])
(get-in request [:headers "accept"])
```

Make the response body look like:

```text
Host: localhost:8080
User-Agent: curl/...
Accept: */*
```

---

## Required tests

```bash
curl -i http://localhost:8080/headers
curl -i -A 'StudentAgent' http://localhost:8080/headers
curl -i -H 'Accept: text/plain' http://localhost:8080/headers
```

---

## Questions

1. Why do we use `get-in` instead of plain `get`?
2. What key contains all request headers?
3. Are header names under `:headers` strings or keywords here?
4. Which command changed the user-agent?
5. Which command changed the accept header?

---

## Stretch

If a header is missing, show:

```text
missing
```

Hint:

```clojure
(or (get-in request [:headers "some-header"]) "missing")
```

---

## Finish line

Explain this:

> `:headers` is a map inside the request map.
