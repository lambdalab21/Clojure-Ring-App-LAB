# 2A. The REPL Is a Remote Control — Feynman Student Lesson

## Why this lesson exists

Before starting and stopping a web server from the REPL, you need one simple idea:

> The REPL is not just a calculator.  
> The REPL is a remote control connected to a running Clojure program.

A beginner mistake is to think development always means this:

```text
edit file
compile program
start program
stop program
edit file again
compile again
start again
```

Clojure lets you work differently:

```text
start a REPL
load code into the running program
change one function
try it immediately
keep the program alive
```

That is the workflow you are learning.

---

## Explain it to a 10-year-old

Imagine a video game is already running.

You do not restart the whole game every time you change your character's name.
You open a command box, type a new name, and the game uses the new name.

The REPL is that command box.

In Clojure, you can send new definitions into a running program:

```clojure
(defn greeting []
  "Hello")
```

Then you can call it:

```clojure
(greeting)
```

Then you can change it:

```clojure
(defn greeting []
  "Hello from the REPL")
```

Then call it again:

```clojure
(greeting)
```

You did not restart the whole program. You changed a piece of it while it was alive.

---

## Technical words after the simple idea

| Simple idea | Clojure word |
|---|---|
| command box connected to a running program | REPL |
| sending code into the running program | evaluation |
| changing a function while the program is alive | redefining a Var |
| checking one small piece at a time | interactive development |

REPL means:

```text
Read → Eval → Print → Loop
```

It reads your code, evaluates it, prints the result, and waits for the next thing.

---

## Part 1 — Try changing a function without restarting anything

In a CIDER REPL, type this:

```clojure
(defn greeting []
  "Hello")
```

Now call it:

```clojure
(greeting)
```

Expected result:

```clojure
"Hello"
```

Now redefine the function:

```clojure
(defn greeting []
  "Hello from the running program")
```

Call it again:

```clojure
(greeting)
```

Expected result:

```clojure
"Hello from the running program"
```

### Check your understanding

Answer before moving on.

1. Did you restart Clojure when you changed `greeting`? No. 
2. What changed: the whole program, or one function definition? Only one function definition(greeting). 
3. Why is this faster than compile → run → stop → edit → compile → run? You avoid full restarts and compile cycles. Changes take effect quickly in the running program. 

---

## Part 2 — A handler is perfect for REPL practice

From the earlier lesson, you know this:

> A Ring handler is just a function.

So you can call it directly in the REPL.

```clojure
(defn my-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body "Hello from handler"})
```

Call it:

```clojure
(my-handler {})
```

Expected result:

```clojure
{:status 200
 :headers {"Content-Type" "text/plain"}
 :body "Hello from handler"}
```

Now make it use the request map:

```clojure
(defn my-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "You asked for: " (:uri request))})
```

Call it with a fake request:

```clojure
(my-handler {:uri "/pizza"})
```

Expected result:

```clojure
{:status 200
 :headers {"Content-Type" "text/plain"}
 :body "You asked for: /pizza"}
```

### Feynman check

Say this out loud:

> I can test a web handler without a web server because the handler is just a function that receives a map and returns a map.

If that sentence does not make sense, stop. Review the handler lesson before touching Jetty.

---

## Part 3 — Why do we need an atom later?

When Jetty starts, it returns a server object.

You need to keep that object somewhere so you can stop it later.

Explain it simply:

> Starting Jetty is like starting a fan.  
> If I want to turn the fan off later, I need to keep the remote control.

In the next lesson, the “remote control” will be stored in an atom.

An atom is a little box that can safely hold changing state.

Try this in the REPL:

```clojure
(defonce box (atom nil))
```

Look inside the box:

```clojure
@box
```

Expected result:

```clojure
nil
```

Put something into the box:

```clojure
(reset! box "server object would go here")
```

Look again:

```clojure
@box
```

Expected result:

```clojure
"server object would go here"
```

### What do these mean?

| Code | Plain meaning |
|---|---|
| `(atom nil)` | make an empty box |
| `@box` | look inside the box |
| `(reset! box x)` | replace what is inside the box with `x` |
| `defonce` | define this only once; do not overwrite it when the file reloads |

---

## Part 4 — Why `defonce` matters

Try this:

```clojure
(defonce box (atom nil))
(reset! box "I want to keep this")
@box
```

Now evaluate this again:

```clojure
(defonce box (atom nil))
```

Check the box:

```clojure
@box
```

The value should still be:

```clojure
"I want to keep this"
```

That is why server code often uses `defonce`:

```clojure
(defonce server (atom nil))
```

If you reload the file, you do not want to forget the running server object.

---

## Part 5 — Mini quiz

Answer without looking back.

1. What does REPL stand for? Read-Evaluate-Print-Loop.
2. In plain English, what does the REPL let you do? Sending new code into a running program and see results without restarting.  
3. Why can you test a Ring handler without Jetty? Because a handler is a regular function that t akes a request map and returns a response map. 
4. What does `@box` do? Dereferences the atom to get its current value. 
5. What does `reset!` do? It replaces the value inside the atom with a new value. 
6. Why might server code use `defonce` instead of `def`? So that reloading the namespace does not lose the reference to the running server object. 
7. What is the danger of only typing commands without predicting the result first? You won't understand what's happening or catch mistakes early. 

---

## Exit ticket

Before going to the next lesson, explain this in your own words:

> How is REPL-driven development different from running the program again and again from the terminal?
REPL-driven developments keep the program running. You send small code changes into the live program and test them instantly instead of restarting from the terminal every time. 

Good answer should mention:

- the program is already running
- you send new code into it
- you test small pieces immediately
- you do not restart everything for every small change

If you cannot explain that, do not continue yet.
