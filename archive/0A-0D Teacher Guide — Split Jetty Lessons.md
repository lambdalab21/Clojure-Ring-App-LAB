# Teacher Guide — Split Jetty Lessons

## Recommendation

The original single lesson is too large for one sitting.

It contains at least five separate concepts:

1. Jetty vs Ring adapter vs handler
2. Raw HTTP vs Ring request map
3. URI vs query string
4. Handler branching and response maps
5. Middleware

For a beginner, teaching all five at once creates the appearance of progress but encourages shallow recognition.

Split it.

---

## Recommended order

```text
0A. Big Picture — Jetty, Adapter, Handler
↓
0B. From Raw HTTP to Ring Request Map
↓
0C. Paper Handler Simulation — No Server Yet
↓
1A. Handler First
↓
1A.5 Handler Review Gate
↓
1B. Jetty Server
↓
2A. The REPL Is a Remote Control
↓
3. Development Workflow
↓
Later: Middleware
```

---

## Should middleware be introduced here?

No, not as a real lesson.

At this point, middleware should be only a preview sentence:

> Later, you will learn that middleware works because handlers are ordinary functions.

Do not teach `wrap-debug` deeply yet unless the student already has a strong command of handlers.

Middleware depends on understanding that:

```text
handler in → handler out
```

That idea is elegant, but if introduced too early it becomes another phrase the student repeats without understanding.

---

## Lesson 0A goal

The student should be able to say:

> Jetty talks to the network and parses HTTP. The Ring adapter converts Jetty's Java request into Clojure data. The handler receives a request map and returns a response map.

Move on only if the student can explain the restaurant analogy and then translate it into technical language.

---

## Lesson 0B goal

The student should understand:

- raw HTTP is not the Ring request map
- `:uri` and `:query-string` are separate
- Jetty parses HTTP before the Ring adapter creates the map

Required checkpoint:

For `/search?q=clojure`, the student must answer:

```clojure
:uri "/search"
:query-string "q=clojure"
```

If the student says `"/search?q=clojure"` for `:uri`, repeat 0B.

---

## Lesson 0C goal

The student should understand:

- a handler can be reasoned about without Jetty
- a handler chooses a response based on request data
- a 404 can be application-generated
- a response map commonly has `:status`, `:headers`, and `:body`

Required checkpoint:

The student should trace a fake request map through the handler without running code.

---

## Red flags

Do not move on if the student says:

- “Jetty is the Clojure framework.”
- “The handler parses HTTP.”
- “The request map is the raw HTTP request.”
- “`:uri` includes the query string.”
- “404 always comes from Jetty.”
- “Middleware is server magic.”

---

## Oral quiz

Ask these after 0A-0C:

1. What does Jetty do?
2. What does the Ring adapter do?
3. What does the handler receive?
4. What does the handler return?
5. What is the difference between raw HTTP and a Ring request map?
6. For `/todos?id=3`, what is `:uri`?
7. For `/todos?id=3`, what is `:query-string`?
8. Who chooses the 404 in the paper handler example?
9. What three keys should a basic Ring response map contain?
10. Why should we practice with fake maps before starting Jetty?

---

## Minimum passing answer

The student may move forward only when he can say something close to this:

> Jetty receives the network request and parses HTTP. The Ring Jetty adapter turns Jetty's Java request object into a Clojure request map. My handler is a normal Clojure function. It reads the request map and returns a response map with status, headers, and body. The adapter and Jetty turn that response map back into an HTTP response.

Do not require perfect wording. Require correct boundaries.
