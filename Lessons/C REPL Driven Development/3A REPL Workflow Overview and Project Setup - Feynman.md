# 3A. REPL Workflow Overview and Project Setup — Feynman Lesson

## Purpose

This lesson answers one question:

> Why do Clojure developers like working from the REPL?

Do **not** try to master server control yet. First understand the workflow.

---

## 1. Explain it to a 10-year-old

Imagine a restaurant.

If the cook changes one recipe, the restaurant should not close, throw away the kitchen, rebuild the building, and reopen.

The manager should be able to say:

> Use this new recipe now.

In Clojure, the REPL is like the manager's walkie-talkie.

You can talk to the running program.

Simple version:

> REPL-driven development means changing and testing small pieces of a running program without restarting everything.

---

## 2. The normal beginner workflow in many languages

A common workflow is:

```text
edit file
compile or run whole program
wait
test
stop
edit again
repeat
```

For web development, beginners often do this:

```text
edit code
stop server
restart server
refresh browser
hope it worked
```

That works, but it makes the server feel like a black box.

---

## 3. The Clojure REPL workflow

The Clojure workflow can be:

```text
start program once
change one function
evaluate that function
test immediately
repeat
```

For a Ring/Jetty web app:

```text
start Jetty from the REPL
change handler
evaluate handler
curl or refresh browser
observe result
```

Feynman version:

> The server keeps running. You update the recipe while the kitchen stays open.

---

## 4. What this lesson sequence will teach

You will learn this in smaller steps:

1. Create a tiny project.
2. Write a handler.
3. Store the running Jetty server object.
4. Start and stop Jetty from the REPL.
5. Use CIDER to evaluate code.
6. Change handler code without restarting Jetty.
7. Learn when restart is actually needed.

Do not jump ahead. The point is not speed. The point is control.

---

## 5. Create the project

From the terminal:

```bash
lein new app sample
cd sample
```

Feynman version:

> Leiningen creates a project folder so the code, dependencies, and startup rules live in one organized place.

---

## 6. Add the dependency

Open `project.clj`.

Use:

```clojure
:dependencies [[org.clojure/clojure "1.11.1"]
               [ring/ring-jetty-adapter "1.15.4"]]
```

Do not add routing, middleware, or reload libraries yet.

More libraries right now would create more places to hide confusion.

---

## 7. Check your understanding

Answer without looking above.

1. What does REPL-driven development mean?
2. Why is it better than restarting the whole app after every small change?
3. What command creates the project?
4. What file lists dependencies?
5. Which dependency lets us run a Ring handler on Jetty?
6. Why are we not adding routing or middleware yet?

---

## 8. Exit ticket

Explain this in 3 sentences:

1. What the REPL lets you do.
2. Why that is useful for web development.
3. What project and dependency you created.

If you cannot explain it, do not continue.
