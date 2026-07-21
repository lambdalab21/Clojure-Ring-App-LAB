# 3E. Break/Fix Drills and Workflow Quiz

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

1. Does the body still appear? Yes, the body still appears. 
2. Does the `Content-Type` header behave as expected? No, content-type header is missing. 
3. Why is `:headers` the correct Ring key? :headers is the correct ring key
4. What did this teach you about response maps? Response map must follow Ring spec, incorrect keys are ignored. 

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

1. Did the running server immediately use the new handler? No. 
2. What changed when you passed `my-handler` instead of `#'my-handler`? It passed a snapshot of the function instead of a live reference. 
3. Which version supports the beginner REPL workflow better? #'my-handler version.
4. Why? It allows live code reloading in the REPL workflow. 

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

1. What does the first call return? :server-started
2. What does the second call return? :server-already-running 
3. Why is that better than trying to start a second server? Prevents port conflicts. 
4. What error might happen if the code allowed two servers on the same port? "Address already in use". 


## 5. The workflow to memorize

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

## 6. Final quiz

Answer without looking.

1. Why is Clojure pleasant for this workflow? Because of immutable data, live reloading, and REPL integration. 
2. What does `cider-jack-in` do? cider-jack-in starts nREPL servers and connects Emacs to the Clojure project. 
3. What does `C-c C-k` do? Loads the entire current file into the REPL. 
4. What does `C-M-x` do? Evaluates the current top-level form. 
5. What does `:join? false` do? Runs Jetty in the background so that REPL stays responsive. 
6. Why do we store the Jetty server object in an atom? To be able to .stop it later. 
7. Why do we use `defonce` for the server atom? Defonce keeps the atom across namespace reloads. 
8. What does `#'my-handler` mean in this lesson? #;my-handler is a var reference. Jetty sees code changes without restarting. 
9. When do you need to restart Jetty? Restarts are needed for server configs. 
10. When can you avoid restarting Jetty? Avoiding restart for handler logic/body changes. 
11. Why is `:headers` plural? :headers is plural because it is a map of multiple headers. 
