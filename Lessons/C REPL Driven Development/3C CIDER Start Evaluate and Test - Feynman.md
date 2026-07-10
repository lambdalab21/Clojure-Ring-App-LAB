# 3C. CIDER: Start, Evaluate, and Test — Feynman Lesson

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
---

## 3. Load the file into the REPL

In the `core.clj` buffer, run:

```text
C-c C-k
```

This loads the file into the running REPL process.

### Check your understanding

1. What does `cider-jack-in` start? It starts an nREPL server connected to your Clojure project and opens a REPL buffer in Emacs. 
2. Do you also need to run `lein repl` manually? No. 
3. What does `C-c C-k` do? It evaluates the current file into the running REPL process.
4. Why does the REPL not know about `start` before the file is loaded? The functions are defined in the source file; they must be evaluated before the REPL can see them. 
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

1. Who receives the HTTP request first: Jetty or your handler?Jetty. 
2. Who calls your handler? The ring jetty adapter. 
3. Why does `/pizza` appear in the response body? The 'my-handler' function reads (:uri request) and includes it in the response body. 
4. What does `curl -i` show that plain `curl` hides? HTTP response headers. 
5. Is the REPL running inside the same program that started Jetty? Yes, the REPL and the web server run in the same JVM process. 

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

1. Which test is faster for checking handler logic? Direct handler call. 
2. Which test proves Jetty and the adapter are also working? The curl test through the real server.  
3. Why should you know both methods? Direct calls are fast for unit-testing logic. Real HTTP tests verify the full stack. 

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

1. What does `(stop)` do to Jetty? Calls on the Jetty Instance. 
2. What does `(stop)` do to the `server` atom? It resets everything to nil. 
3. Why should curl fail after stopping the server? No process is listening on port 8080 anymore. 
4. What does `(restart)` do? Calls (stop) then (start)
