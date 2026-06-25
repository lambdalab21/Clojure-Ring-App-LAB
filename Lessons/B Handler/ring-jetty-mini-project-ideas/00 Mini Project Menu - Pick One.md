# Mini Project Menu — Ring Handler + Jetty

## Purpose

Pick **one** small project.

Do not try to build a real web app yet.

These projects use only what you have learned:

```text
HTTP request
↓
Jetty + Ring adapter
↓
Ring request map
↓
handler
↓
Ring response map
↓
HTTP response
```

No Hiccup.  
No Reitit.  
No middleware.  
No database.  
No frontend framework.

The point is to prove that you understand handlers, request maps, response maps, and Jetty.

---

## Required starting point

Use your existing `jetty-demo` project from the Jetty lesson.

You should already have:

```clojure
(ns jetty-demo.core
  (:require [ring.adapter.jetty :as jetty])
  (:gen-class))
```

and you should know how to run:

```bash
lein run
```

Then test with:

```bash
curl -i http://localhost:8080/
```

---

## Project choices

| Project | Main skill |
|---|---|
| Project 1: Request Inspector | Read request-map keys |
| Project 2: Three-Page Manual Router | Route with `:uri` |
| Project 3: Method Responder | Route with `:request-method` |
| Project 4: Query String Detective | Understand `:uri` vs `:query-string` |
| Project 5: Status Code Practice Server | Return different status codes |
| Project 6: Header Inspector | Read request headers |
| Project 7: Tiny Text API Without JSON Library | Return API-like plain text |
| Project 8: Broken Server Detective | Debug common beginner mistakes |

Pick one easy project first. Then pick a second one.

---

## Rules

1. Predict before running `curl`.
2. Use `curl -i`, not only browser refresh.
3. Write down what request-map key the handler reads.
4. Write down what response-map keys the handler returns.
5. Explain the project in plain English when finished.

If you cannot explain it, you did not finish. You only typed.

---

## Minimum completion checklist

Your project is complete only when you can answer:

1. What URL did you test?
2. What request-map key did your handler inspect?
3. What response map did your handler return?
4. What did `curl -i` show?
5. What mistake did you almost make?
