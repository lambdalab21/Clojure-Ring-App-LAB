# Teacher Guide — 0B and 0C HTTP/Ring Bridge

## Main recommendation

Do not add code to 0A.

0A should stay conceptual:

```text
Jetty → adapter → handler
```

The missing bridge is:

```text
HTTP request → Ring request map
Ring response map → HTTP response
```

That should be its own lesson before the first coding lab.

Recommended sequence:

```text
0A. Big Picture — Jetty, Adapter, Handler
0B. HTTP Messages to Ring Maps — Feynman Bridge
0C. First Coding Lab — See HTTP Become Ring Maps
0D. Paper Handler Simulation — No Server Yet
1A. Handler First
1A.5 Handler Review Gate
1B. Jetty Server
2A. The REPL Is a Remote Control
3. Development Workflow
```

## Should students code at this point?

Yes, but not in 0A.

Use this split:

- **0A:** no code; build the mental picture.
- **0B:** no project; paper mapping between HTTP and Ring maps.
- **0C:** first tiny coding lab using `lein new app`.

This prevents the student from learning commands before understanding what the commands are supposed to prove.

## Why 0B is needed

Without 0B, students may parrot:

> handler receives a request map and returns a response map

but not understand how that relates to actual HTTP.

0B forces them to connect:

```text
GET /search?q=clojure HTTP/1.1
```

with:

```clojure
{:request-method :get
 :uri "/search"
 :query-string "q=clojure"}
```

and:

```clojure
{:status 200
 :headers {"Content-Type" "text/plain"}
 :body "Hello"}
```

with:

```http
HTTP/1.1 200 OK
Content-Type: text/plain

Hello
```

## Why 0C should include project creation

At 0C, yes, include project creation:

```bash
lein new app http-map-demo
cd http-map-demo
```

Then add the Ring Jetty adapter dependency.

The student should see a complete path:

```text
create project → add dependency → write handler → run server → send HTTP request → inspect response
```

Do not use CIDER or REPL-driven development yet in 0C. That comes later.

Reason:

- `lein run` is boring but simple.
- CIDER is powerful but adds editor, REPL, namespace, evaluation, and process-control concepts.
- The student should first understand HTTP/Ring maps.

## Dependency choice

Use:

```clojure
[ring/ring-jetty-adapter "1.15.4"]
```

This is narrower and clearer than adding the broad `[ring "..."]` dependency when the lesson is specifically about the Jetty adapter.

## What counts as success for 0B

The student can:

- explain that HTTP is outside-program communication
- explain that Ring maps are inside-Clojure data
- identify `:uri` and `:query-string`
- explain that `:status`, `:headers`, and `:body` describe the response
- explain that the response map is not literally the raw HTTP response

## What counts as success for 0C

The student can:

- create the project without guessing
- explain why the Jetty adapter dependency is needed
- run the server
- use `curl` to send URLs
- predict `:uri` and `:query-string` before running commands
- explain why `curl -i` shows HTTP response headers and status
- explain that the handler chose the 404 in the deliberate 404 experiment

## Answers for 0B prediction drill

| URL | `:uri` | `:query-string` |
|---|---|---|
| `http://localhost:3000/` | `/` | `nil` |
| `http://localhost:3000/about` | `/about` | `nil` |
| `http://localhost:3000/search?q=clojure` | `/search` | `q=clojure` |
| `http://localhost:3000/todos?id=3&done=false` | `/todos` | `id=3&done=false` |
| `http://localhost:3000/users/42` | `/users/42` | `nil` |

## Oral questions before moving on

1. What is the difference between an HTTP request and a Ring request map?
2. What is the difference between a Ring response map and an HTTP response?
3. Why does the handler not need to parse raw HTTP text?
4. In the 404 experiment, who chose the 404?
5. What did `curl -i` reveal?

## Warning signs

Do not move on if the student says vague phrases like:

- “Jetty runs the code.”
- “The map is the request.”
- “The server makes the 404.”
- “The dependency makes HTTP work.”
- “I just copied the handler.”

Force the student to use precise wording:

> Jetty receives and parses HTTP. The Ring adapter gives my handler a Ring request map. My handler returns a Ring response map.
