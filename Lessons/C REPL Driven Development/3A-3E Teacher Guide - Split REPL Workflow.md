# Teacher Guide — Split REPL-Driven Web Development Lessons

## Why the original lesson should be split

The original lesson is useful, but it tries to teach too much at once:

- project setup
- dependency setup
- handler review
- server atom
- `start`, `stop`, `restart`
- CIDER commands
- `curl` testing
- `#'my-handler`
- restart rules
- deliberate break/fix exercises
- final quiz

That is not one beginner lesson. It is a small module.

The student may appear to understand because he can copy the code and run it. That is not enough.

---

## Use this order

```text
3A. REPL Workflow Overview and Project Setup
↓
3B. REPL-Controlled Server Code
↓
3C. CIDER: Start, Evaluate, and Test
↓
3D. Change Code Without Restarting Jetty
↓
3E. Break/Fix Drills and Workflow Quiz
```

---

## What each lesson should prove

### 3A

The student understands the purpose of REPL-driven development.

Must be able to say:

> The REPL lets me change and test a running program without restarting everything.

---

### 3B

The student understands the code before using CIDER.

Must be able to explain:

- handler
- server atom
- `start`
- `stop`
- `restart`
- `:join? false`
- `#'my-handler`

Do not let him continue if he cannot explain the server atom.

---

### 3C

The student can operate CIDER and Jetty.

Must be able to:

- run `cider-jack-in`
- load the buffer with `C-c C-k`
- call `(start)`
- test with `curl`
- call `(stop)`

---

### 3D

The student experiences the main Clojure payoff.

Must be able to change handler behavior without restarting Jetty.

Key sentence:

> Jetty can use the new handler because it was started with the handler Var, `#'my-handler`.

---

### 3E

The student proves understanding through break/fix drills.

Do not skip this. It prevents fake confidence.

---

## Warning signs

The student is not ready to move on if he says:

- “CIDER restarts the server automatically.”
- “The REPL is just a terminal.”
- “The atom is the server.”
- “`#'my-handler` is just syntax decoration.”
- “I always restart after every change.”
- “`C-c C-k` and `C-M-x` are basically the same.”

Correct these immediately.

---

## Strong understanding checklist

The student can:

- explain REPL-driven development simply
- start and stop Jetty from the REPL
- explain why `:join? false` matters
- explain why the server object is stored
- evaluate a single handler function
- test using curl
- explain when restart is and is not needed
- explain `#'my-handler` with the recipe-book analogy
- recover from common mistakes

---

## Recommended next lesson

After this module, the next good topic is manual routing or a tiny route table.

Do not introduce middleware yet unless handler and REPL workflow are automatic.
