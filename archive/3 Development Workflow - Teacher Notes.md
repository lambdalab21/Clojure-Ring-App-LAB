# 3. REPL-Driven Web Development — Teacher Notes and Answer Key

## Purpose of this revised lesson

The original file introduces a valuable idea: use Emacs, CIDER, and `cider-jack-in` so the student can start, stop, and restart the web app from the REPL instead of repeatedly using `lein run`. It also correctly points toward the REPL-controlled workflow. However, it mixes too many ideas at once and contains several errors that would confuse a beginner.

This revised version splits the learning path into:

1. **2A. The REPL Is a Remote Control** — bridge lesson before web-server control.
2. **3. REPL-Driven Web Development with Jetty** — actual Jetty + CIDER workflow.

The student should not touch Lesson 3 until he can explain Lesson 2A.

---

## Problems corrected from the original file

### 1. Command-line setup was compressed incorrectly

Original idea:

```bash
lein new app sample cd sample
```

Correct beginner version:

```bash
lein new app sample
cd sample
```

Do not combine these in a teaching file. The student needs to know that the first command creates the project and the second command enters it.

---

### 2. Namespace mismatch

The project is named `sample`, so the namespace should usually be:

```clojure
(ns sample.core
  ...)
```

The original used:

```clojure
(ns req-res.core ...)
```

That mismatch is unnecessary noise for a beginner.

---

### 3. Wrong Ring response key

Original code used:

```clojure
:header {"Content-Type" "text/html"}
```

Correct Ring response key:

```clojure
:headers {"Content-Type" "text/plain"}
```

Ring expects `:headers`, plural.

---

### 4. Port mismatch

The original code starts Jetty on `8080` but later says the app is running on `8081`.

The revised lesson uses `8080` consistently.

---

### 5. `server` vs `my-server` mismatch

The original defines:

```clojure
(defonce my-server (atom nil))
```

but later talks about `server`.

The revised lesson uses one name consistently:

```clojure
(defonce server (atom nil))
```

---

### 6. `wrap-reload` was mentioned but not actually used

The original says:

> Because you’re using `wrap-reload` with `#'my-handler`...

But the shown code does not use `wrap-reload`.

The revised beginner lesson does **not** introduce `wrap-reload`. It uses `#'my-handler` and manual CIDER evaluation. That is simpler and more honest.

Later, after the student understands REPL evaluation, you can teach `wrap-reload` as a convenience.

---

## Key idea the student must learn

The student must be able to say:

> The REPL is connected to the running JVM. I can evaluate new definitions into that running process. If Jetty uses a live Var reference like `#'my-handler`, then changing and evaluating the handler lets the running server use the new handler without restarting Jetty.

If he cannot say that, he is just copying commands.

---

## Recommended order

```text
Understanding Jetty
↓
1A Handler First
↓
1A.5 Handler Review Gate
↓
1B Jetty Server
↓
2A The REPL Is a Remote Control
↓
3 REPL-Driven Web Development with Jetty
```

---

## Answer key — Lesson 2A mini quiz

### 1. What does REPL stand for?

Read, Eval, Print, Loop.

### 2. In plain English, what does the REPL let you do?

It lets you send code into a running Clojure process, run it, inspect results, and redefine parts of the program without restarting everything.

### 3. Why can you test a Ring handler without Jetty?

Because a Ring handler is just a function that takes a request map and returns a response map.

### 4. What does `@box` do?

It dereferences the atom and returns the current value inside it.

### 5. What does `reset!` do?

It replaces the value inside an atom.

### 6. Why might server code use `defonce` instead of `def`?

Because reloading the file should not recreate the atom and lose the reference to the running server object.

### 7. What is the danger of only typing commands without predicting the result first?

The student may create the illusion of progress while not building a mental model. Prediction forces understanding.

---

## Answer key — Lesson 3 checks

### Part 1

1. `lein new app sample` creates a new Leiningen application project named `sample`.
2. `project.clj` controls dependencies.
3. The Jetty adapter dependency gives the project `ring.adapter.jetty/run-jetty`.
4. Avoid mixing Ring versions casually. Use one consistent version for beginner lessons.

### Part 3

1. `my-handler` is a normal function.
2. It receives a request map.
3. It returns a response map.
4. It uses `(:uri request)`.

### Part 4

1. We store the server object so we can stop it later.
2. `@server` means “look inside the atom.”
3. `defonce` prevents the server atom from being recreated when the file is reloaded.

### Part 5

1. `start` checks `@server` to avoid starting duplicate servers.
2. Starting twice on the same port can produce an “Address already in use” error.
3. `:join? false` lets the REPL keep accepting input.
4. `#'my-handler` passes a live Var reference so the server can use newer evaluated versions of the handler.

### Part 6

1. `stop` sets the atom back to `nil` so the program knows no server is running.
2. Calling `(stop)` twice should return `:server-not-running` the second time.
3. `(restart)` calls `(stop)` and then `(start)`.

### Part 7

1. No. `cider-jack-in` starts the REPL for the project.
2. `C-c C-k` loads the current buffer into the REPL process.
3. The REPL cannot call functions that have not been loaded/evaluated.

### Part 8

1. Jetty receives the HTTP request first.
2. The Ring Jetty adapter calls the handler.
3. `/pizza` appears because Jetty builds a request map whose `:uri` is `"/pizza"`, and the handler uses `(:uri request)`.
4. Direct handler call skips Jetty and HTTP; `curl` sends a real HTTP request through Jetty.

### Part 9

1. Jetty did not restart.
2. Jetty used the new handler because `#'my-handler` points to the current Var value.
3. `C-M-x` evaluates one top-level form.
4. `C-c C-k` reloads the whole buffer.

### Part 10

1. No restart usually needed for changing only the handler body.
2. Yes, restart needed for changing the port.
3. Handler body is looked up through the Var. Port is part of server setup captured when Jetty starts.

---

## Answer key — Lesson 3 quiz

1. REPL-driven development means changing and testing a running program through the REPL.
2. Clojure is pleasant for this because functions and Vars can be redefined interactively in the running process.
3. `cider-jack-in` starts a project-connected CIDER REPL.
4. `C-c C-k` loads the current buffer.
5. `C-M-x` evaluates the top-level form at point.
6. `:join? false` starts Jetty without blocking the REPL.
7. The atom stores the running server object so it can be stopped.
8. `defonce` prevents the atom from being replaced during reloads.
9. `#'my-handler` is a Var reference to the current handler definition.
10. Restart Jetty when server setup changes.
11. Avoid restarting when only handler logic changes and Jetty was given `#'my-handler`.
12. It is not a typing exercise because the student must predict, explain, test directly, test through HTTP, and compare results.

---

## What to watch for

A student who understands will naturally say things like:

- “The handler is just a function.”
- “Jetty is already running.”
- “I evaluated the new handler into the running REPL.”
- “The server atom stores the Jetty object.”
- “I only need restart when I change server setup.”

A student who is copying will say things like:

- “I pressed C-c C-k because the instruction said so.”
- “I don’t know why restart is needed here.”
- “I changed the code, but I don’t know whether Jetty restarted.”
- “I don’t know what `#'` does.”

Stop there and return to the earlier sections.
