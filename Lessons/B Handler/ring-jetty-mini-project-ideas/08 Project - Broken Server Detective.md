# Project 8 — Broken Server Detective

## Goal

Practice finding common beginner mistakes.

This project strengthens disciplined debugging.

---

## Rule

Do not fix by guessing.

Use this process:

```text
read the error or symptom
predict the cause
change one thing
test again
explain the fix
```

---

## Broken example 1 — Wrong response key

Buggy response:

```clojure
{:status 200
 :header {"Content-Type" "text/plain"}
 :body "Hello"}
```

Questions:

1. What key is wrong?
2. What should it be?
3. Why might the body still appear?
4. Why does `curl -i` help?

---

## Broken example 2 — Misspelled require

Buggy namespace:

```clojure
(ns jetty-demo.core
  (:requre [ring.adapter.jetty :as jetty])
  (:gen-class))
```

Questions:

1. What is misspelled?
2. What should it be?
3. Would this fail before or after the server starts?

---

## Broken example 3 — Handler ignores request

```clojure
(defn handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body "Same every time"})
```

A student expects `/about` to show about-page text.

Questions:

1. Why does every URL return the same body?
2. What request-map key should the handler read?
3. How could `case` help?

---

## Broken example 4 — Address already in use

Symptom:

```text
Address already in use
```

Questions:

1. What does this mean?
2. What port is probably already occupied?
3. What command can show the process using the port?
4. Why should `kill -9` not be the first habit?

---

## Broken example 5 — URI/query confusion

A student writes:

```clojure
(case (:uri request)
  "/search?q=clojure" ...)
```

But this does not match.

Questions:

1. Why not?
2. What is `:uri`?
3. What is `:query-string`?
4. What should the route probably match?

---

## Finish line

Pick two broken examples.

For each one, write:

1. Symptom
2. Cause
3. Fix
4. Lesson learned
