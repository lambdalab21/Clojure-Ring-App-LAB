# Clojure Ring/Jetty Lesson File Order

## Short answer

Use the files in this order:

1. `0A. Big Picture — Jetty, Adapter, Handler.md`
2. `0B HTTP Messages to Ring Maps - Feynman Bridge.md`
3. `0C Paper Handler Simulation - No Server Yet.md`
4. `0C First Coding Lab -See HTTP Become Ring Maps.md`
5. `0D Middleware Preview - Save for Later.md` — do **not** teach yet; keep only as a later preview.

## What to keep

### 1. `0A. Big Picture — Jetty, Adapter, Handler.md`

Use first.

Purpose:

- Big mental picture.
- No coding.
- Student learns the roles of Jetty, Ring adapter, handler, request map, and response map.

This lesson should answer:

> Who does what before my Clojure function runs?

Do not expand this into code. Keep it simple.

---

### 2. `0B HTTP Messages to Ring Maps - Feynman Bridge.md`

Use second.

Purpose:

- Connect outside HTTP messages to inside Ring maps.
- Covers both directions:
  - HTTP request → Ring request map
  - Ring response map → HTTP response

This is better than the older `0B. From Raw HTTP to Ring Request Map.md` because it covers both request and response.

This lesson should answer:

> What do request maps and response maps have to do with real HTTP requests and responses?

No project yet. This is still paper-and-brain work.

---

### 3. `0C Paper Handler Simulation - No Server Yet.md`

Use third.

Purpose:

- Practice `request map → handler → response map`.
- No Jetty.
- No `lein`.
- No server.
- No browser.

This prevents fake understanding. The student must prove he understands what a handler does before using a real server.

This lesson should answer:

> If a handler receives this request map, what response map will it return?

Recommended rename:

`0C Paper Handler Simulation - No Server Yet.md`

Keep the name if you want. It is fine.

---

### 4. `0C First Coding Lab -See HTTP Become Ring Maps.md`

Use fourth, but rename it to avoid duplicate numbering.

Recommended rename:

`0D First Coding Lab - See HTTP Become Ring Maps.md`

Purpose:

- First actual coding lesson.
- Create a Leiningen project.
- Add the Ring Jetty adapter dependency.
- Run a tiny Jetty/Ring app.
- Use `curl` to see real request data.

This lesson should answer:

> Can I send a real HTTP request and see what Ring gives my handler?

This is where coding starts.

---

### 5. `0D Middleware Preview - Save for Later.md`

Do not teach as a real lesson yet.

Recommended rename:

`Later Middleware Preview - Save for Later.md`

Purpose:

- Preserve the idea for later.
- Maybe give one sentence only:
  > Middleware works because handlers are ordinary functions.

Do not make the student work through middleware yet.

Use it later after:

- handler-first lesson
- Jetty server lesson
- REPL workflow lesson
- maybe basic manual routing

## What to archive or disregard

### Archive: `0B. From Raw HTTP to Ring Request Map.md`

Reason:

- It is not wrong.
- But it is now superseded by `0B HTTP Messages to Ring Maps - Feynman Bridge.md`.
- The newer bridge is better because it connects both:
  - HTTP request → Ring request map
  - Ring response map → HTTP response

Do not use both. That would create repetition and confusion.

## Clean recommended sequence

Use this sequence:

```text
0A. Big Picture — Jetty, Adapter, Handler
↓
0B. HTTP Messages to Ring Maps — Feynman Bridge
↓
0C. Paper Handler Simulation — No Server Yet
↓
0D. First Coding Lab — See HTTP Become Ring Maps
↓
1A. Handler First — Feynman Student Lesson
↓
1A.5 Handler Review Gate — Before Jetty
↓
1B. Jetty Server — Feynman Student Lesson
↓
2A. The REPL Is a Remote Control — Feynman Student Lesson
↓
3. Development Workflow — Feynman Student Lesson
↓
Later: Middleware
```

## Should students code yet?

Not in 0A.

Not in 0B.

Not in paper simulation.

Start coding in the first coding lab.

Reason:

A beginner needs three separate mental steps:

1. Who are the actors?  
   Jetty, adapter, handler.

2. What data moves across the boundary?  
   HTTP request/response outside, Ring maps inside.

3. What does the handler do with the maps?  
   Request map in, response map out.

Only after those three are stable should the student create a project with:

```bash
lein new app http-map-demo
```

and add:

```clojure
[ring/ring-jetty-adapter "1.15.4"]
```

## Middleware decision

Do not teach middleware seriously yet.

Middleware belongs later.

At this stage, the student only needs this preview:

> Middleware works because a Ring handler is just a function that receives a request map and returns a response map.

If the student cannot explain handlers clearly, middleware will become fake vocabulary.
