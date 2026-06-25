# End-of-Chapter Test — Teacher Answer Key
## Chapter: Jetty, Ring Maps, and Handler Simulation

Use this to grade Versions A-D.

Do not give this to the student before the test.

---

# Version A — Answer Key

## Part 1

### 1. Matching

| Term | Answer |
|---|---|
| Jetty | B |
| Ring Jetty adapter | D |
| Ring handler | E |
| Ring request map | A |
| Ring response map | C |

### 2. Flow

Accept answers close to:

```text
curl/browser
↓
network connection
↓
Jetty parses HTTP
↓
Ring Jetty adapter creates Ring request map
↓
handler receives the request map
↓
handler returns Ring response map
↓
Ring Jetty adapter and Jetty create the final HTTP response
```

### 3.

The handler does not read directly from the socket. Jetty receives network bytes and parses HTTP. The Ring adapter gives the handler a Clojure request map.

---

## Part 2

| URL | `:uri` | `:query-string` |
|---|---|---|
| `http://localhost:3000/` | `"/"` | `nil` |
| `http://localhost:3000/about` | `"/about"` | `nil` |
| `http://localhost:3000/search?q=clojure` | `"/search"` | `"q=clojure"` |
| `http://localhost:3000/todos?id=3&done=false` | `"/todos"` | `"id=3&done=false"` |
| `http://localhost:3000/users/42` | `"/users/42"` | `nil` |

Accept blank/empty for no query string only if the student explains that Ring commonly represents this as `nil`.

---

## Part 3

1. `404`
2. `Content-Type: text/plain`
3. `Not found`
4. The handler could choose this 404.

---

## Part 4

### Request A

1. `"/about"`
2. `"/about"` branch
3. `200`
4. `"About"`

### Request B

1. `"/missing"`
2. Default branch
3. `404`
4. The handler chose it.

---

## Part 5

Strong answer includes:

- Jetty receives/parses HTTP.
- Ring adapter converts to request map.
- Handler receives request map.
- Handler returns response map.
- Adapter/Jetty convert response map to HTTP response.
- No raw socket work in handler.

---

# Version B — Answer Key

## Part 1

| Restaurant story | Web development idea |
|---|---|
| Customer | Browser, curl, HTTPie, client |
| Messy order | HTTP request |
| Front desk worker | Jetty |
| Translator | Ring Jetty adapter |
| Clean order ticket | Ring request map |
| Cook | Handler |
| Finished tray | Ring response map |

## Part 2

Accept close corrections.

1. Jetty is the web server; the handler is a function that decides a response.
2. Request map is Clojure data made from parsed HTTP.
3. `:uri` is the path; `:query-string` is separate.
4. The handler can return a 404 response map.
5. The response map is Clojure data that adapter/Jetty turn into an HTTP response.

## Part 3

```clojure
{:request-method :get
 :uri "/books/7"
 :query-string "format=json"
 :headers {"host" "localhost:3000"
           "user-agent" "TestClient"
           "accept" "*/*"}}
```

## Part 4

1. `200`
2. Body is JSON
3. String
4. Client may use it to parse/display body correctly.

## Part 5

1. `:uri`
2. `"/"`
3. No
4. `"Home"`
5. Handler must inspect `:query-string` and branch on it.

## Part 6

1. Server/request path
2. Handler logic
3. Handler/Ring map misunderstanding; inspect request map meaning
4. Jetty
5. Handler

---

# Version C — Answer Key

## Part 1

Definitions should be simple and accurate.

Key points:

- Jetty: Java server; listens/parses/writes response.
- Adapter: bridge between Jetty Java objects and Ring maps.
- Request map: Clojure map describing parsed request.
- Handler: function from request map to response map.
- Response map: Clojure map describing response.
- HTTP response: actual message sent back to client.

## Part 2

Accept this order:

```text
curl/browser
↓
Jetty receives connection
↓
Jetty parses HTTP
↓
Java servlet request object
↓
Ring Jetty adapter
↓
Ring request map
↓
handler
↓
Ring response map
↓
Ring Jetty adapter / Jetty
↓
HTTP response
```

## Part 3

A. `:uri "/search"`, `:query-string "q=clojure"`  
B. `:uri "/users/42"`, `:query-string "tab=settings"`  
C. `:uri "/about"`, `:query-string nil`

## Part 4

A.

```clojure
{:status 200
 :headers {"Content-Type" "text/plain"}
 :body "About"}
```

B.

```clojure
{:status 404
 :headers {"Content-Type" "text/plain"}
 :body "Not found"}
```

C.

```clojure
{:status 200
 :headers {"Content-Type" "text/plain"}
 :body "Home"}
```

The handler does not care about `:request-method`.

## Part 5

1. `:status`
2. `:headers`
3. `:body`
4. No
5. Ring adapter and Jetty

## Part 6

Correct explanation:

The path is `:uri`; the part after `?` is `:query-string`.

---

# Version D — Answer Key

## Part 1

Strong answer includes:

- curl sends HTTP request.
- Jetty receives it.
- Jetty parses HTTP.
- Adapter creates Ring request map.
- Handler reads request map.
- Handler returns response map.
- Adapter/Jetty send HTTP response.

## Part 2

Request map:

```clojure
{:request-method :get
 :uri "/products/12"
 :query-string "view=short"
 :headers {"host" "localhost:3000"}}
```

Possible response map:

```clojure
{:status 200
 :headers {"Content-Type" "text/plain"}
 :body "any plain text"}
```

## Part 3

| Job | Responsible part |
|---|---|
| Listens on port 3000 | Jetty |
| Parses HTTP syntax | Jetty |
| Converts Java servlet request to Ring request map | Ring adapter |
| Receives request map | Handler |
| Decides body text `"About"` | Handler |
| Returns response map | Handler |
| Converts Ring response map toward Java response | Ring adapter |
| Writes final bytes back to client | Jetty |

## Part 4

Accept:

```clojure
(defn handler [request]
  (case (:uri request)
    "/"        {:status 200
                :headers {"Content-Type" "text/plain"}
                :body "Home"}

    "/about"  {:status 200
                :headers {"Content-Type" "text/plain"}
                :body "About"}

    "/contact" {:status 200
                 :headers {"Content-Type" "text/plain"}
                 :body "Contact page"}

    {:status 404
     :headers {"Content-Type" "text/plain"}
     :body "Not found"}))
```

## Part 5

1. `:header` is wrong.
2. Correct key is `:headers`.
3. Ring expects response headers under `:headers`.

## Part 6

Better explanation:

Jetty receives/parses HTTP. The Ring adapter creates the request map and calls the handler. The handler returns a response map, not raw HTTP. The adapter and Jetty turn the response map into an HTTP response.

## Part 7

1. request map → response map
2. request
3. response
4. Clojure data / Ring maps
5. handler
