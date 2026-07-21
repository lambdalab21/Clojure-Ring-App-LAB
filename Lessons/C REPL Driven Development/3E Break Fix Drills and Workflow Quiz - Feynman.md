# 3E. Break/Fix Drills and Workflow Quiz — Feynman Lesson

## Purpose

You will deliberately break small things and explain what happened.

Do this only after Lessons 3A-3D work.

---

## 1. Break/fix rule

Do not randomly break the program.

Use this process:

```text
predict
break one thing
evaluate or restart only when appropriate
test
explain
fix
test again
```

---

## 2. Break 1: Wrong response key

Change this correct response key:

```clojure
:headers {"Content-Type" "text/plain"}
```

to this wrong key:

```clojure
:header {"Content-Type" "text/plain"}
```

Evaluate the handler:

```text
C-M-x
```

Test:

```bash
curl -i http://localhost:8080/
```

Questions:

1. Does the body still appear?
2. Does the `Content-Type` header behave as expected?
3. Why is `:headers` the correct Ring key?
4. What did this teach you about response maps?

Fix it before continuing.

---

## 3. Break 2: Pass function value instead of Var

In `start`, temporarily change:

```clojure
(jetty/run-jetty #'my-handler
                 {:port 8080
                  :join? false})
```

to:

```clojure
(jetty/run-jetty my-handler
                 {:port 8080
                  :join? false})
```

Now reload the whole file and restart:

```text
C-c C-k
```

```clojure
(restart)
```

Change the text inside `my-handler`.

Evaluate only the handler:

```text
C-M-x
```

Test with curl.

Questions:

1. Did the running server immediately use the new handler?
2. What changed when you passed `my-handler` instead of `#'my-handler`?
3. Which version supports the beginner REPL workflow better?
4. Why?

Fix it:

```clojure
#'my-handler
```

Then reload and restart.

---

## 4. Break 3: Start twice

Call:

```clojure
(start)
(start)
```

Questions:

1. What does the first call return?
2. What does the second call return?
3. Why is that better than trying to start a second server?
4. What error might happen if the code allowed two servers on the same port?

---

## 5. Break 4: Stop twice

Call:

```clojure
(stop)
(stop)
```

Questions:

1. What does the first call return?
2. What does the second call return?
3. Why should stopping an already-stopped server not crash the learning workflow?

---

## 6. The workflow to memorize

For handler changes:

```text
edit handler
C-M-x or C-c C-k
curl or refresh browser
observe
```

For server setup changes:

```text
edit server setup
C-c C-k
(restart)
curl or refresh browser
observe
```

For debugging handler logic:

```text
call handler directly with fake request map
observe response map
then test through Jetty
```

---

## 7. Final quiz

Answer without looking.

1. What is REPL-driven development?
2. Why is Clojure pleasant for this workflow?
3. What does `cider-jack-in` do?
4. What does `C-c C-k` do?
5. What does `C-M-x` do?
6. What does `:join? false` do?
7. Why do we store the Jetty server object in an atom?
8. Why do we use `defonce` for the server atom?
9. What does `#'my-handler` mean in this lesson?
10. When do you need to restart Jetty?
11. When can you avoid restarting Jetty?
12. Why is `:headers` plural?
