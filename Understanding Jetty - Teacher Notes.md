# Understanding Jetty — Teacher Notes and Answer Key

## Main teaching goal

The student should stop thinking of the web server as magic and should understand the boundary:

```text
HTTP / Java servlet world → Ring adapter → Clojure data/function world
```

The most important sentence they should be able to say:

> Jetty handles sockets and HTTP parsing; the Ring adapter turns the parsed Java servlet request into a Ring request map; my handler receives that map and returns a response map.

---

## What changed from the original note

The original note had the correct core idea, but it was mostly explanatory. The revised version turns it into a guided lesson with:

- clearer separation between Jetty, Ring adapter, and handler
- repeated request/response flow diagrams
- prediction questions before running code
- small code experiments
- middleware connection
- common mistakes
- final review questions
- a short written explanation task
- an exit ticket

This makes passive copy/paste harder because the student must predict, run, compare, explain, and modify.

---

## Expected answers: Section 1

1. Jetty listens on a port, accepts sockets, parses HTTP, manages connections/threads, and writes response bytes.
2. The adapter translates from Jetty Java servlet objects to Ring Clojure maps, and from Ring response maps back to Java servlet responses.
3. A Ring request map.
4. A Ring response map.
5. The handler receives parsed Clojure data; Jetty and the adapter handled the socket and parsing work before the handler was called.

---

## Expected answers: Section 2

1. Jetty HTTP parser.
2. Ring Jetty adapter.
3. At `(handler request)`.
4. When the handler returns the response map.
5. Jetty rejects malformed HTTP before Ring or the handler sees it.

---

## Expected answers: Section 3

1. No. It is a Clojure representation of a parsed HTTP request.
2. Jetty.
3. Because Jetty has already parsed and accepted the request before the adapter creates the Ring map.
4. `:uri` is the path, such as `/todos`; `:query-string` is the part after `?`, such as `id=3`.
5. Headers are grouped under the `:headers` key because the request map organizes related request metadata.

---

## Expected answers: Section 4

For:

```bash
curl 'http://localhost:3000/todos?id=3'
```

Expected values:

1. `:get`
2. `/todos`
3. `id=3`
4. `/todos`
5. `id=3`

Watch for this mistake: students often think `:uri` includes the query string. In Ring, it does not.

---

## Expected answers: Section 5

1. It starts an embedded Jetty server.
2. It prevents the REPL thread from being blocked by the server.
3. It lets the running server use the latest definition after the handler is redefined.
4. The running server may keep using the old function value until restarted.
5. The student can change code, test quickly, and build a mental model through fast feedback.

---

## Expected answers: Section 6

1. In `[:headers "user-agent"]`.
2. Because `:headers` is a nested map inside the request map.
3. It may return `nil` unless a default is provided.
4. Jetty parsed the raw header.
5. The Ring Jetty adapter placed the parsed header into the Ring request map.

---

## Expected answers: Section 7

1. `:uri`.
2. The `case` expression falls through to the default response.
3. The handler is deciding that `/missing` should return 404.
4. Response headers and status line.
5. Ring responses need to tell the adapter what HTTP status code to send.

---

## Expected answers: Section 8

1. A handler function.
2. A new handler function.
3. Inside the returned anonymous function, after printing the URI.
4. Middleware operates at the Ring function/map level, not inside Jetty.
5. Each middleware is just a function that takes a handler and returns a handler.

---

## Final review answer checklist

A good answer should include these points:

- Jetty is the Java server.
- Jetty handles sockets, connections, and HTTP parsing.
- Ring adapter bridges Jetty and Ring.
- The adapter creates the Ring request map.
- The adapter calls the handler.
- The handler is a Clojure function.
- The handler receives a request map.
- The handler returns a response map.
- A basic response map has `:status`, `:headers`, and `:body`.
- Middleware works because handlers are ordinary functions.
- Malformed HTTP does not normally reach the handler because Jetty rejects it first.

---

## Grading rubric

Use this to check whether the student really understood the lesson.

### Strong understanding

The student can:

- draw the complete request/response flow without notes
- explain the role of Jetty, adapter, and handler separately
- predict `:request-method`, `:uri`, and `:query-string`
- modify the handler without copying blindly
- explain why middleware is function composition
- explain why malformed HTTP does not reach the handler

### Partial understanding

The student can run the code and identify some request-map keys but still mixes up:

- Jetty vs Ring adapter
- raw HTTP vs Ring request map
- URI path vs query string
- server-generated 404 vs application-generated 404

### Weak understanding

The student can copy the code but cannot answer:

- who calls the handler
- what the handler receives
- what the handler returns
- where HTTP parsing happens
- why middleware works

Do not move on if the student is here.

---

## Suggested oral questions

Ask these verbally after the student completes the file:

1. Explain what happens after `curl` connects to port 3000.
2. Where does the request stop being raw bytes?
3. Where does the request become Clojure data?
4. If `/missing` returns 404, who chose that status code in this example?
5. Why is middleware possible without changing Jetty?
6. What would you inspect if the handler prints the wrong URI?
7. What would you inspect if the handler never runs?

---

## Follow-up mini-lab

Have the student add one middleware function:

```clojure
(defn wrap-request-log [handler]
  (fn [request]
    (println (:request-method request) (:uri request))
    (handler request)))
```

Then run:

```clojure
(def app
  (wrap-request-log handler))

(run-jetty #'app {:port 3000 :join? false})
```

They must answer:

1. Is `app` still a handler?
2. Does Jetty know that middleware exists?
3. Which function receives the request first: `wrap-request-log`'s returned function or `handler`?
4. Why does the final response still come from `handler`?

Expected key idea:

> Middleware returns another handler, so Jetty/Ring can treat the wrapped application exactly like a normal handler.
