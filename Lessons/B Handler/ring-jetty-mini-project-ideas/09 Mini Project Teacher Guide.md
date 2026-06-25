# Mini Project Teacher Guide — Ring Handler + Jetty

## Purpose

These projects come after:

- Handler First
- Handler Review Gate
- Jetty Server

They should be used before adding middleware, routing libraries, Hiccup, static files, or reload helpers.

The goal is not to build impressive apps.

The goal is to test whether the student really understands:

```text
request map → handler → response map
```

and how Jetty brings real HTTP requests to the handler.

---

## Recommended project order

If the student wants a safe path, use this order:

1. Request Inspector
2. Three-Page Manual Router
3. Method Responder
4. Query String Detective
5. Status Code Practice Server
6. Header Inspector
7. Tiny Text API
8. Broken Server Detective

If the student chooses only one, recommend:

- Request Inspector for request-map understanding
- Three-Page Manual Router for URI/routing understanding
- Broken Server Detective for discipline

---

## Minimum passing standard

The student must be able to explain:

1. What request-map key the project reads.
2. What response-map keys the project returns.
3. What `curl -i` proved.
4. Whether Jetty, the adapter, or the handler made the decision.
5. Why the project did not need an extra library.

---

## Warning signs

The student is faking understanding if he says:

- “Jetty routes the pages.”
- “The handler receives the URL string.”
- “`:uri` is `/path?query=value`.”
- “The response map is the same thing as raw HTTP.”
- “I changed the dependency to add a page.”
- “I used the browser, so I know the status code.”

Correct immediately.

---

## Grading rubric

### Strong

The student can:

- predict before testing
- use `curl -i`
- explain request map and response map
- add a route without changing Jetty setup
- explain `:uri` vs `:query-string`
- explain who chose a 404

### Partial

The student can run the project but mixes up:

- Jetty vs handler responsibility
- URI vs query string
- response body vs response status
- request headers vs response headers

### Weak

The student can copy code but cannot explain:

- what the handler receives
- what the handler returns
- why `curl -i` matters
- why no new library was needed

Do not move on to middleware or routing libraries if the student is weak here.
