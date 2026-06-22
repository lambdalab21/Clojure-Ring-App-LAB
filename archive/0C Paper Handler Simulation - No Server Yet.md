# 0C. Paper Handler Simulation — No Server Yet

## Purpose

This lesson answers one question:

> If the handler receives a request map, how does it decide the response map?

No Jetty. No server. No browser. No copy/paste escape route.

Just maps and functions.

---

## 1. Feynman version

The cook receives a clean order ticket.

The cook reads one important line:

> What page did the customer ask for?

Then the cook returns the correct tray.

In Ring terms:

```text
request map → handler → response map
```

---

## 2. The handler

Read this handler slowly.

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

Plain English:

> Look at the request's `:uri`. If it is `/`, return Home. If it is `/about`, return About. Otherwise return Not found.

---

## 3. Simulation 1

Pretend this request map goes into the handler:

```clojure
{:request-method :get
 :uri "/about"
 :query-string nil
 :headers {"host" "localhost:3000"}}
```

Answer before running anything:

1. What is `(:uri request)`?
2. Which branch of `case` runs?
3. What status comes back?
4. What body comes back?
5. Did Jetty choose this response, or did the handler choose it?

---

## 4. Simulation 2

Now use this request map:

```clojure
{:request-method :get
 :uri "/missing"
 :query-string nil
 :headers {}}
```

Answer:

1. What is `(:uri request)`?
2. Which branch of `case` runs?
3. What status comes back?
4. What body comes back?
5. Who chose the 404 in this example?

Important lesson:

> A 404 can come from your application handler. Jetty is not always the one deciding “not found.”

---

## 5. Simulation 3

Use this request map:

```clojure
{:request-method :get
 :uri "/"
 :query-string "debug=true"
 :headers {"user-agent" "curl"}}
```

Answer:

1. What is `(:uri request)`?
2. Does the handler care about `:query-string` in this version?
3. Which response comes back?
4. What would you need to change if you wanted `debug=true` to matter?

---

## 6. Response map shape

A basic Ring response map usually has:

```clojure
{:status 200
 :headers {"Content-Type" "text/plain"}
 :body "Hello"}
```

For now, remember these three keys:

| Key | Meaning |
|---|---|
| `:status` | HTTP status code to send back |
| `:headers` | response metadata |
| `:body` | response content |

Feynman version:

> The response map is the finished tray: status label, extra notes, and the actual food.

---

## 7. Mini quiz

1. What key does this handler use to choose the page?
2. What does status `200` mean in this lesson?
3. What does status `404` mean in this lesson?
4. Is `:headers` singular or plural?
5. Why is `:header` wrong in a normal Ring response map?
6. Why is it useful to simulate the handler before starting a server?

---

## 8. Explain-back task

Close the file. Explain this in 5 sentences:

1. What a request map is.
2. What a handler does with the request map.
3. What a response map is.
4. How this handler chooses between `/`, `/about`, and missing pages.
5. Why a 404 can be chosen by your handler.

---

## 9. Exit ticket

Fill in the blanks:

1. A Ring handler is a Clojure __________.
2. It receives a __________ map.
3. It returns a __________ map.
4. The handler above branches on the request's __________.
5. In this example, `/missing` returns status __________.

If you cannot answer these from memory, do not move to the Jetty server lesson.
