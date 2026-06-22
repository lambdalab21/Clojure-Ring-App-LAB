# 3C. CIDER: Start, Evaluate, and Test — Feynman Lesson

## Purpose

This lesson answers one question:

> How do I use Emacs + CIDER to control the running web server?

You already wrote the server-control code. Now you will use it from the REPL.

---

## 1. Open the project

Open this file in Emacs:

```text
src/sample/core.clj
```

Make sure it contains the code from Lesson 3B.

---

## 2. Start CIDER

Run:

```text
M-x cider-jack-in
```

Feynman version:

> CIDER starts a REPL that is connected to your project.

You do **not** need to run `lein repl` manually when using `cider-jack-in`.

---

## 3. Load the file into the REPL

In the `core.clj` buffer, run:

```text
C-c C-k
```

This loads the file into the running REPL process.

Feynman version:

> Loading the file is like telling the running program, “Here are the functions I want you to know.”

After this, the REPL should know:

```clojure
my-handler
start
stop
restart
server
```

### Check your understanding

1. What does `cider-jack-in` start?
2. Do you also need to run `lein repl` manually?
3. What does `C-c C-k` do?
4. Why does the REPL not know about `start` before the file is loaded?

---

## 4. Start the server from the REPL

In the REPL, run:

```clojure
(start)
```

Expected result:

```clojure
:server-started
```

Run it again:

```clojure
(start)
```

Expected result:

```clojure
:server-already-running
```

This is good. Your `start` function is protecting you from starting two servers on the same port.

---

## 5. Test with curl

Open another terminal.

Run:

```bash
curl -i http://localhost:8080/
```

Expected shape:

```text
HTTP/1.1 200 OK
Content-Type: text/plain

Hello from the REPL.
URI: /
```

Try another URI:

```bash
curl -i http://localhost:8080/pizza
```

Expected body:

```text
Hello from the REPL.
URI: /pizza
```

### Check your understanding

1. Who receives the HTTP request first: Jetty or your handler?
2. Who calls your handler?
3. Why does `/pizza` appear in the response body?
4. What does `curl -i` show that plain `curl` hides?
5. Is the REPL running inside the same program that started Jetty?

---

## 6. Direct handler test versus real HTTP test

You can test the handler directly:

```clojure
(my-handler {:uri "/pizza"})
```

You can also test through Jetty:

```bash
curl -i http://localhost:8080/pizza
```

These are different.

Direct handler call:

```text
fake request map → handler → response map
```

Curl through Jetty:

```text
HTTP request → Jetty → adapter → request map → handler → response map → adapter → Jetty → HTTP response
```

Both are useful.

### Check your understanding

1. Which test is faster for checking handler logic?
2. Which test proves Jetty and the adapter are also working?
3. Why should you know both methods?

---

## 7. Stop and restart

In the REPL:

```clojure
(stop)
```

Expected:

```clojure
:server-stopped
```

Try curl again:

```bash
curl -i http://localhost:8080/
```

It should fail because the server is stopped.

Start again:

```clojure
(start)
```

Then test again with curl.

### Check your understanding

1. What does `(stop)` do to Jetty?
2. What does `(stop)` do to the `server` atom?
3. Why should curl fail after stopping the server?
4. What does `(restart)` do?

---

## 8. Exit ticket

Explain this without looking:

> CIDER starts a REPL connected to my project. I load my file into that REPL. Then I can call `(start)` to start Jetty, use `curl` to send real requests, and call `(stop)` to stop the server.

If you cannot explain it, repeat this lesson.
