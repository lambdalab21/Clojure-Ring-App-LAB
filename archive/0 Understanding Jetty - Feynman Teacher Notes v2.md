# Understanding Jetty — Feynman Teacher Notes and Answer Key

## Main teaching goal

The student should understand the boundary between the outside HTTP world and the Clojure function world.

Target sentence:

> Jetty talks to the network and parses HTTP. The Ring Jetty adapter converts Jetty's Java servlet request into a Clojure request map. My handler receives that map and returns a response map.

Do not let the student move on if he only says:

> I run Jetty and it shows a page.

That is too shallow.

---

## Why this Feynman revision exists

The previous version was technically useful, but still too textbook-like. It explained the layers correctly, but a student could still skim the diagrams and pretend to understand.

This version forces the student through this order:

```text
simple story → technical words → prediction → paper simulation → explain back
```

That is the point. Copying commands is not the goal.

---

## Recommended position in the sequence

Use this as the first conceptual lesson:

```text
0 Understanding Jetty, Ring, and Handler
↓
1A Handler First
↓
1A.5 Handler Review Gate
↓
1B Jetty Server
↓
2A The REPL Is a Remote Control
↓
3 Development Workflow
```

This file should not become a coding race. It is a mental-model lesson.

---

## Section 0 expected answers: 10-year-old picture

1. Customer = browser, `curl`, or HTTPie.
2. Front desk worker = Jetty.
3. Translator = Ring Jetty adapter.
4. Cook = Ring handler.
5. Clean order ticket = Ring request map.
6. Finished tray = Ring response map / HTTP response being prepared.

Watch for weak answers:

- “Jetty is the handler.” Wrong.
- “The handler receives HTTP.” Too vague.
- “The request map is the website.” Wrong.

Correct them immediately.

---

## Section 1 expected answers: real names

1. Jetty listens on a port, accepts connections, reads raw bytes, parses HTTP, manages connection details, and writes response bytes back.
2. The adapter translates between Jetty's Java servlet objects and Ring's Clojure maps.
3. The handler receives a Ring request map and returns a Ring response map.
4. The handler does not directly manage sockets; Jetty does that work before Ring calls the handler.
5. The two maps are the request map and response map.

Good student wording:

> Jetty is outside-world plumbing. The handler is Clojure decision logic.

---

## Section 2 expected explanation

A good 3-5 sentence explanation should include:

- A client such as `curl` sends an HTTP request.
- Jetty receives and parses the request.
- The adapter turns Jetty's Java request into a Clojure request map.
- The handler receives the map and returns a response map.
- The adapter/Jetty send the response back.

Weak answer pattern:

> Jetty sends the request to Clojure and Clojure sends back a page.

This is not wrong enough to fail, but it is too vague. Ask:

- What exactly does Clojure receive?
- Which Clojure function receives it?
- What exactly does that function return?

---

## Section 3 expected answers: URI and query string

| URL | `:uri` | `:query-string` |
|---|---|---|
| `http://localhost:3000/` | `/` | `nil` |
| `http://localhost:3000/about` | `/about` | `nil` |
| `http://localhost:3000/search?q=clojure` | `/search` | `q=clojure` |
| `http://localhost:3000/todos?id=3&done=false` | `/todos` | `id=3&done=false` |

Important correction:

Students often put `/todos?id=3&done=false` under `:uri`. Do not accept that. Make them repeat:

> `:uri` is the path. `:query-string` is the part after the question mark.

---

## Section 4 expected answers: valid HTTP

1. Jetty checks whether the HTTP syntax is valid.
2. No. The handler normally does not receive raw malformed HTTP text.
3. Because Jetty and the adapter have already done the lower-level HTTP/server work.
4. If the handler never runs, inspect server/Jetty/adapter/startup/port issues first.
5. If the handler runs but returns the wrong body, inspect handler logic and the request map values.

Teaching note:

This is a good place to teach debugging boundaries:

```text
No request reaches handler → server/listener/routing/startup problem
Handler runs but wrong output → handler/data/logic problem
```

---

## Section 5 expected one-sentence drill

Accept simple sentences like:

- Jetty is the server that talks to the network.
- The Ring Jetty adapter turns Jetty's Java request into a Clojure map.
- A Ring handler is a Clojure function that decides the response.
- A request map is Clojure data describing the client's request.
- A response map is Clojure data describing what the server should send back.

Reject vague sentences like:

- “Jetty runs stuff.”
- “Adapter adapts things.”
- “Handler handles.”

Those are word games, not understanding.

---

## Section 6 expected answers: paper simulation

For:

```clojure
{:request-method :get
 :uri "/about"
 :query-string nil
 :headers {"host" "localhost:3000"}}
```

Expected answers:

1. `(:uri request)` is `"/about"`.
2. The `"/about"` branch runs.
3. `:status` is `200`.
4. `:body` is `"About"`.
5. The handler chose this response.

For:

```clojure
{:request-method :get
 :uri "/missing"
 :query-string nil
 :headers {}}
```

Expected answers:

1. Status `404`.
2. Body `"Not found"`.
3. The handler chose the 404 in this example.

Important distinction:

Jetty may also generate its own errors in some situations, but in this lesson's code, the application handler chooses the 404.

---

## Section 7 expected answers: middleware

1. `wrap-debug` receives a handler.
2. It returns a new handler function.
3. No. Jetty does not need to know middleware exists.
4. Because the wrapped app has the same shape: request map in, response map out.
5. The middleware wrapper prints the URI.
6. The original handler returns the final response, unless the middleware changes it.

Key phrase to listen for:

> Middleware takes a handler and returns a handler.

If the student cannot say that, do not move on to routing libraries.

---

## Section 8 common wrong ideas

Use these corrections during oral review.

### If the student says:

> Jetty calls my route.

Ask:

- What is the Ring adapter doing?
- Does Jetty know Clojure route functions directly?

Better answer:

> Jetty receives and parses HTTP. The Ring adapter calls the Ring handler.

---

### If the student says:

> My handler parses the HTTP request.

Ask:

- Does the handler receive raw HTTP text?
- What data structure does it receive?

Better answer:

> The handler receives a Ring request map.

---

### If the student says:

> 404 means Jetty could not find the page.

Ask:

- In our code, who returned `{:status 404 ...}`?

Better answer:

> In this example, the handler chose to return 404.

---

## Section 9 final Feynman explanation rubric

A strong answer should include all of these:

- Client sends HTTP request.
- Jetty receives network traffic.
- Jetty parses HTTP.
- Adapter converts Java servlet request to Ring request map.
- Handler receives request map.
- Handler returns response map.
- Adapter/Jetty convert/send the response.
- Middleware works by wrapping handlers.

Partial answer:

- Mentions Jetty and handler but not adapter.
- Says “request” but not “request map.”
- Says “response” but not “response map.”

Weak answer:

- Explains only commands or tools.
- Cannot distinguish Jetty from handler.
- Cannot explain where Clojure data appears.

Do not proceed if the answer is weak.

---

## Section 10 exit ticket answers

1. Jetty handles **sockets/network connections** and **HTTP parsing**.
2. The Ring adapter changes Jetty's Java request into a **Ring request map**.
3. A handler is a Clojure **function**.
4. A handler receives a **request map** and returns a **response map**.
5. Middleware works because handlers are ordinary **functions**.

Passing standard:

- 5/5: move on.
- 4/5: correct the miss, then move on.
- 3/5 or below: repeat the lesson.

---

## Suggested oral quiz

Ask these without letting the student look at the file:

1. Explain the restaurant analogy.
2. Now explain the same idea using the words Jetty, adapter, handler.
3. What does the handler receive?
4. What does the handler return?
5. Where does HTTP parsing happen?
6. What is the difference between `:uri` and `:query-string`?
7. If `/missing` returns 404 in our handler, who chose that?
8. Why can middleware wrap a handler?
9. What would you inspect if the handler never runs?
10. What would you inspect if the handler runs but gives the wrong response?

---

## Follow-up mini-lab for review

Do this before Lesson 1A if the student is shaky.

Give the student this fake request map:

```clojure
{:request-method :get
 :uri "/profile"
 :query-string "user=elliot"
 :headers {"user-agent" "curl"}}
```

Ask him to write the response map by hand for this handler:

```clojure
(defn handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "You asked for " (:uri request))})
```

Expected response:

```clojure
{:status 200
 :headers {"Content-Type" "text/plain"}
 :body "You asked for /profile"}
```

Then ask:

1. Did Jetty choose the body?
2. Did the adapter choose the body?
3. What part of the system chose the body?
4. What part of the request map did the handler read?

Expected key idea:

> The handler chose the response body by reading `:uri` from the request map.

---

## Teacher warning

Do not let the student rush this lesson.

The danger sign is not slow typing. The danger sign is fast typing with no explanation.

Make him pause at these moments:

- after the restaurant analogy
- after the full request path
- after the URI/query-string table
- after the paper simulation
- before moving to actual Jetty code

The point is to build the mental model before the tool use.
