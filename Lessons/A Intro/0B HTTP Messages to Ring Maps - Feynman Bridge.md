# 0B. HTTP Messages to Ring Maps — Feynman Bridge

## Purpose

This lesson connects two worlds:

```text
HTTP request  →  Ring request map
Ring response map  →  HTTP response
```

Do **not** create a Clojure project yet.

This is a paper-and-brain lesson. The goal is to understand what the maps mean before writing more code.

If you rush into code now, you will probably copy commands and tell yourself you understand. That is weak learning.

---

## 1. Explain it to a 10-year-old

Imagine a customer sends a messy order to a restaurant.

The front desk rewrites it into a clean order ticket for the cook.

The cook does not read the customer's original messy words. The cook reads the clean ticket.

Then the cook gives back a finished tray with:

- a result label
- some notes
- the food

Web version:

| Restaurant story | Web idea |
|---|---|
| Messy customer order | HTTP request |
| Clean order ticket | Ring request map |
| Cook | Handler |
| Finished tray | Ring response map |
| Tray sent back to customer | HTTP response |

Simple version:

> HTTP is what travels outside. Ring maps are what your Clojure code works with inside.

---

## 2. HTTP request → Ring request map

A raw HTTP request may look like this:

```http
GET /search?q=clojure HTTP/1.1
Host: localhost:3000
User-Agent: curl/8.x
Accept: */*
```

Your handler does **not** receive that raw text.

Ring gives your handler a map shaped like this:

```clojure
{:request-method :get
 :uri "/search"
 :query-string "q=clojure"
 :headers {"host" "localhost:3000"
           "user-agent" "curl/8.x"
           "accept" "*/*"}}
```

Feynman version:

> The HTTP request is the outside message. The Ring request map is the clean Clojure version of that message.

---

## 3. The most important request-map keys for now

You do not need every Ring request key yet. Learn these first.

| Ring key | Meaning | Example |
|---|---|---|
| `:request-method` | HTTP method | `:get` |
| `:uri` | path part of the URL | `"/search"` |
| `:query-string` | part after `?` | `"q=clojure"` |
| `:headers` | request headers | `{"host" "localhost:3000"}` |

Important mistake to avoid:

```text
/search?q=clojure
```

is split into:

```clojure
:uri "/search"
:query-string "q=clojure"
```

Do **not** say the URI is `"/search?q=clojure"` in the Ring request map.

---

## 4. Prediction drill: URL → request-map pieces

Fill this out before checking anything in code.

| URL                                           | `:uri` | `:query-string` |
| --------------------------------------------- | ------ | --------------- |
| `http://localhost:3000/`                      |        |                 |
| `http://localhost:3000/about`                 |        |                 |
| `http://localhost:3000/search?q=clojure`      |        |                 |
| `http://localhost:3000/todos?id=3&done=false` |        |                 |
| `http://localhost:3000/users/42`              |        |                 |

### Explain

1. Which part becomes `:uri`?
2. Which part becomes `:query-string`?
3. What should `:query-string` be when there is no `?` in the URL?

---

## 5. Ring response map → HTTP response

Your handler returns a Ring response map:

```clojure
{:status 200
 :headers {"Content-Type" "text/plain"}
 :body "Hello"}
```

The adapter and Jetty turn that into an HTTP response roughly like this:

```http
HTTP/1.1 200 OK
Content-Type: text/plain

Hello
```

Feynman version:

> The response map is the cook's finished tray. Jetty and the adapter turn the tray into something the customer can receive.

---

## 6. The three basic response-map keys

For now, focus on these:

| Ring response key | Meaning | Example |
|---|---|---|
| `:status` | HTTP status code | `200`, `404`, `500` |
| `:headers` | response metadata | `{"Content-Type" "text/plain"}` |
| `:body` | response content | `"Hello"` |

Simple version:

```text
:status  = what happened?
:headers = extra labels about the response
:body    = the actual content
```

---

## 7. Prediction drill: response map → HTTP meaning

For each response map, explain what the client should receive.

### A

```clojure
{:status 200
 :headers {"Content-Type" "text/plain"}
 :body "Home page"}
```

Questions:

1. What status code is sent?
2. What content type is sent?
3. What body text is sent?

---

### B

```clojure
{:status 404
 :headers {"Content-Type" "text/plain"}
 :body "Not found"}
```

Questions:

1. What status code is sent?
2. Who chose this 404 in this example: Jetty or the handler?
3. What body text is sent?

---

### C

```clojure
{:status 200
 :headers {"Content-Type" "application/json"}
 :body "{\"message\":\"ok\"}"}
```

Questions:

1. What does the `Content-Type` claim the body is?
2. Is the body a Clojure map or a string here?
3. Why might the client care about the content type?

---

## 8. Put the full idea together

A handler is a function with this shape:

```text
Ring request map  →  Ring response map
```

Example:

```clojure
(defn handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "You requested " (:uri request))})
```

If the request map is:

```clojure
{:request-method :get
 :uri "/about"
 :query-string nil
 :headers {}}
```

Then the response body should be:

```text
You requested /about
```

### Stop and answer

1. What does the handler receive?
2. What does the handler return?
3. Which key did the handler read from the request map?
4. Which key did the handler build in the response map?
5. Did the handler parse raw HTTP text?

---

## 9. Common wrong ideas to kill early

### Wrong idea 1

> The request map is the HTTP request.

Better:

> The request map is Clojure data made from the parsed HTTP request.

---

### Wrong idea 2

> The response map is the HTTP response.

Better:

> The response map is Clojure data that the adapter and Jetty turn into an HTTP response.

---

### Wrong idea 3

> A 404 must come from Jetty.

Better:

> A handler can choose to return a 404 response map.

---

## 10. Final teach-back

Close the file. Explain this in 5 sentences:

1. What is an HTTP request?
2. What is a Ring request map?
3. What is a handler?
4. What is a Ring response map?
5. What is an HTTP response?

Use this sentence pattern if you are stuck:

> Outside the program, the client sends an HTTP request. Inside the Clojure program, my handler receives a Ring request map. The handler returns a Ring response map. The adapter and Jetty turn that response map into an HTTP response.

---

## 11. Exit ticket

Fill in the blanks from memory.

1. HTTP request travels on the __________ side.
2. Ring request map is Clojure __________.
3. Handler receives a __________ and returns a __________.
4. `:status`, `:headers`, and `:body` belong to the Ring __________ map.
5. `:request-method`, `:uri`, and `:query-string` belong to the Ring __________ map.

If you miss more than one, repeat this lesson before coding.
