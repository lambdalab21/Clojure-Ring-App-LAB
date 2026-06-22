# 4G. Middleware Teacher Guide and Answer Key

## Purpose

This guide helps you check whether the student understands middleware instead of merely typing examples.

## Recommended order

```text
4A. Middleware Is Just a Function
↓
4B. First Middleware — Logging the Request
↓
4C. Middleware Can Change the Response
↓
4D. Middleware Can Change the Request
↓
4E. Middleware Order and Nested Wrapping
↓
4F. Thread Macro for Middleware Pipelines
```

Do not introduce external libraries during this module.

No Hiccup.

No Reitit.

No reload.

No Ring-devel.

No static-file middleware.

The goal is to understand the shape:

```text
handler → handler
```

---

## Key teaching checkpoints

### After 4A

The student must be able to say:

> A handler receives a request map and returns a response map. A middleware receives a handler and returns a new handler.

If not, stop.

---

### After 4B

The student must understand:

- `wrap-log-request` receives a handler.
- It returns a new handler.
- The returned handler prints the URI.
- Then it calls the original handler.
- Jetty can run the wrapped handler because it is still a handler.

---

### After 4C

The student must understand:

- Middleware can call the handler first.
- Middleware can capture the response map.
- Middleware can return a changed response map.
- `assoc-in` is useful for nested maps like `[:headers "X-Powered-By"]`.

---

### After 4D

The student must understand:

- Middleware can create a new request map.
- `assoc` returns a new map.
- The original request is not mutated.
- The handler receives the changed request map.

---

### After 4E

The student must understand:

- Middleware order matters.
- Request goes outside to inside.
- Response goes inside to outside.
- The outer middleware sees the request first and the response last.

---

### After 4F

The student must understand:

- `->` is not middleware.
- `->` is a convenience for writing nested calls.
- Nested version and threaded version can build the same app.
- Middleware works even without the thread macro.

---

## Common fake-understanding phrases

Correct these immediately.

### Wrong

> Middleware is something from a library.

Better:

> Libraries often provide middleware, but middleware itself is just a function that wraps a handler.

### Wrong

> Middleware changes Jetty.

Better:

> Middleware wraps the Ring handler. Jetty only runs the final handler.

### Wrong

> The thread macro makes middleware work.

Better:

> The thread macro only makes nested wrapping easier to read.

### Wrong

> Middleware always runs before the handler.

Better:

> Middleware can do work before the handler, after the handler, or both.

---

## Oral exam questions

Ask these without letting the student look at the file.

1. What is the shape of a handler?
2. What is the shape of middleware?
3. Why is the return value of middleware also a handler?
4. In `wrap-log-request`, who prints the URI?
5. In `wrap-powered-by`, who creates the original response?
6. In `wrap-powered-by`, who changes the response?
7. Does `wrap-add-student-name` mutate the request?
8. In nested middleware, which wrapper sees the request first?
9. Which wrapper sees the response last?
10. What does the thread macro do?
11. Is the thread macro required for middleware?
12. Why are libraries not needed to learn middleware?

---

## Minimum passing explanation

The student may move on when he can say something like:

> A Ring handler is a function from request map to response map. Middleware is a function that receives a handler and returns another handler. The returned handler can inspect or change the request, call the original handler, inspect or change the response, and return the final response. Jetty does not need to know about each middleware; it only runs the final wrapped handler. The thread macro is only a cleaner way to write nested wrapping.

If he cannot say this, do not move to middleware libraries yet.
