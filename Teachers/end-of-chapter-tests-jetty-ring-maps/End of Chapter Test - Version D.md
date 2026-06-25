# End-of-Chapter Test — Version D
## Chapter: Jetty, Ring Maps, and Handler Simulation

### Instructions for the student

This is the hardest version.

It mixes explanation, prediction, and correction.

---

## Part 1 — One-minute oral answer

Prepare a one-minute spoken answer to this question:

> What happens from `curl http://localhost:3000/about` to the final response?

Write bullet points first.

Bullet points:

- 
- 
- 
- 
- 
- 

Now write the answer in paragraph form.

---

## Part 2 — Complete the maps

A client requests:

```text
http://localhost:3000/products/12?view=short
```

Complete the likely request map.

```clojure
{:request-method ______
 :uri ______
 :query-string ______
 :headers {"host" "localhost:3000"}}
```

Now write a possible response map for a successful plain-text response.

```clojure
{:status ______
 :headers {______ ______}
 :body ______}
```

---

## Part 3 — Who is responsible?

For each job, write Jetty, Ring adapter, or handler.

| Job | Responsible part |
|---|---|
| Listens on port 3000 | |
| Parses HTTP syntax | |
| Converts Java servlet request to Ring request map | |
| Receives request map | |
| Decides body text `"About"` | |
| Returns response map | |
| Converts Ring response map toward Java response | |
| Writes final bytes back to client | |

---

## Part 4 — Handler modification reasoning

Original handler:

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

You want `/contact` to return:

```text
Contact page
```

with status `200`.

Write the modified handler.

```clojure

```

---

## Part 5 — Find the bug

A student writes:

```clojure
{:status 200
 :header {"Content-Type" "text/plain"}
 :body "Hello"}
```

Question:

1. What is wrong?
2. What is the correct key?
3. Why does this matter?

---

## Part 6 — Identify fake understanding

A student says:

> I understand. Jetty sends a request map to the handler, and the handler returns HTTP.

Correct this carefully.

Better explanation:

---

## Part 7 — Final no-notes test

Close all lesson files. Answer from memory.

1. A handler has the shape: __________ → __________.
2. `:request-method`, `:uri`, and `:query-string` belong to the __________ map.
3. `:status`, `:headers`, and `:body` belong to the __________ map.
4. The adapter is the boundary between Java servlet objects and __________.
5. A 404 can be chosen by __________.
