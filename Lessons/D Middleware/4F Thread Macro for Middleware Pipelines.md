# 4F. Thread Macro for Middleware Pipelines

## Purpose

This lesson answers one question:

> After I understand nested middleware, how can I write it more clearly?

Now you may learn the thread macro.

Do **not** start here. Start with nested wrapping first.

---

## 1. The nested version

You already wrote something like:

```clojure
(def app
  (wrap-powered-by
    (wrap-log-request
      (wrap-add-student-name
        #'my-handler))))
```

This is correct.

But as more wrappers are added, it becomes harder to read.

---

## 2. Explain it with Bash piping

In Bash, you might write:

```bash
cat access.log | grep ERROR | sort
```

You could describe that as:

```text
take access.log
send it to grep
send the grep result to sort
```

The pipe makes a chain easier to read.

Clojure's thread macro can do something similar for function calls.

---

## 3. The thread-first macro `->`

This:

```clojure
(-> x
    f
    g
    h)
```

means roughly:

```clojure
(h (g (f x)))
```

---

## 4. Convert the middleware nesting

Nested version:

```clojure
(def app
  (wrap-powered-by
    (wrap-log-request
      (wrap-add-student-name
        #'my-handler))))
```

Threaded version:

```clojure
(def app
  (-> #'my-handler
      wrap-add-student-name
      wrap-log-request
      wrap-powered-by))
```

This builds the same wrapped handler.

Read it as:

```text
start with #'my-handler
wrap it with wrap-add-student-name
wrap that with wrap-log-request
wrap that with wrap-powered-by
```

---

## 5. Important warning

The thread macro is only a writing convenience.

Middleware is still just:

```text
handler → handler
```

---

## 6. Compare both versions

Fill in the blanks.

Nested:

```clojure
(def app
  (wrap-c
    (wrap-b
      (wrap-a
        #'my-handler))))
```

Threaded:

```clojure
(def app
  (-> #'my-handler
      ______
      ______
      ______))
```

Answer:

```clojure
(def app
  (-> #'my-handler
      wrap-a
      wrap-b
      wrap-c))
```

---

## 7. Execution order still matters

With:

```clojure
(def app
  (-> #'my-handler
      wrap-add-student-name
      wrap-log-request
      wrap-powered-by))
```

Build order:

```text
my-handler → add-student-name → log-request → powered-by
```

Request enters the final outside wrapper first:

```text
wrap-powered-by
↓
wrap-log-request
↓
wrap-add-student-name
↓
my-handler
```

---

## 8. Check your understanding

1. What does `->` help with? Readability of nested function calls. 
2. Is `->` required for middleware? No. 
3. What does `(-> x f g)` become? (g (f x))
4. Convert `(c (b (a x)))` into thread-macro style. (-> x a b c)
5. Why did we learn nested wrapping before `->`? So that the real wrapping/order is understood first. 
6. Why is “middleware is just a library feature” wrong? Middleware is just ordinary functions that take and return handlers. 
