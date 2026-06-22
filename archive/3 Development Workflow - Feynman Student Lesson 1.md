# 3. REPL-Driven Web Development with Jetty — Feynman Student Lesson

## Goal

You already know two important ideas:

1. A Ring handler is just a function.
2. Jetty can run that handler as a web server.

Now you will learn the development workflow that makes Clojure pleasant:

> Start the server from the REPL, change code, evaluate the new code, refresh the browser, and keep going.

You are not just trying to “make the page work.”
You are learning how to control a running program.

---

## Explain it to a 10-year-old

Imagine a restaurant kitchen.

- The customer sends an order.
- The cook prepares food.
- The waiter brings food back.

In our web app:

| Restaurant | Web app |
|---|---|
| customer order | HTTP request |
| cook | handler function |
| finished meal | HTTP response |
| restaurant building | Jetty server |
| manager's walkie-talkie | REPL |

The REPL is like the manager's walkie-talkie.

You do not close the whole restaurant every time you change one recipe.
You tell the cook, “Use the new recipe now.”

That is REPL-driven development.

---

## Part 1 — Create the project

From the terminal:

```bash
lein new app sample
cd sample
```

Open `project.clj`.

Use a dependency for the Jetty adapter:

```clojure
:dependencies [[org.clojure/clojure "1.11.1"]
               [ring/ring-jetty-adapter "1.15.4"]]
```

If your earlier lessons used a different Ring version, use one Ring version consistently across the project.

### Check your understanding

1. What does `lein new app sample` create?
2. What file controls the project dependencies?
3. Why do we need a Jetty adapter dependency?
4. What should you avoid doing with Ring versions inside the same beginner project?

---

## Part 2 — Write simple REPL-controlled server code

Edit `src/sample/core.clj`:

```clojure
(ns sample.core
  (:require [ring.adapter.jetty :as jetty]))

(defn my-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "Hello from the REPL.\n"
              "URI: " (:uri request) "\n")})

(defonce server (atom nil))

(defn start []
  (if @server
    :server-already-running
    (do
      (reset! server
              (jetty/run-jetty #'my-handler
                               {:port 8080
                                :join? false}))
      :server-started)))

(defn stop []
  (if @server
    (do
      (.stop @server)
      (reset! server nil)
      :server-stopped)
    :server-not-running))

(defn restart []
  (stop)
  (start))
```

Do not rush past this code. Each piece has a job.

---

## Part 3 — Read the code like a human

### `my-handler`

```clojure
(defn my-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "Hello from the REPL.\n"
              "URI: " (:uri request) "\n")})
```

Plain English:

> When a request comes in, return a text response. Also show the URI that the client requested.

Test it before Jetty:

```clojure
(my-handler {:uri "/test"})
```

Expected response map:

```clojure
{:status 200
 :headers {"Content-Type" "text/plain"}
 :body "Hello from the REPL.\nURI: /test\n"}
```

### Check your understanding

1. Is `my-handler` a special web-server object or a normal function?
2. What kind of value does it receive?
3. What kind of value does it return?
4. What part of the request map is used in the response body?

---

## Part 4 — The `server` atom

```clojure
(defonce server (atom nil))
```

Plain English:

> Make a box named `server`. At first, the box is empty. Later, we will put the running Jetty server object into it.

Why?

Because when you start Jetty, you need to keep the server object so you can stop it later.

```clojure
@server
```

means:

> Look inside the server box.

```clojure
(reset! server something)
```

means:

> Replace what is inside the server box.

### Check your understanding

1. Why do we need to store the server object?
2. What does `@server` mean?
3. Why use `defonce` instead of plain `def`?

---

## Part 5 — The `start` function

```clojure
(defn start []
  (if @server
    :server-already-running
    (do
      (reset! server
              (jetty/run-jetty #'my-handler
                               {:port 8080
                                :join? false}))
      :server-started)))
```

Plain English:

> If the server is already running, do not start another one.  
> If it is not running, start Jetty, put the server object in the box, and return `:server-started`.

Important details:

```clojure
:port 8080
```

means:

> Listen on port 8080.

```clojure
:join? false
```

means:

> Start Jetty, but do not trap the REPL forever. Let me keep typing commands.

```clojure
#'my-handler
```

means:

> Give Jetty a live reference to the current `my-handler`, not just an old copy of the function.

That matters. It lets you redefine `my-handler`, reload it, and have Jetty use the newer version.

### Feynman analogy for `#'my-handler`

Passing `my-handler` is like giving Jetty a photocopy of today's recipe.

Passing `#'my-handler` is like giving Jetty the recipe book's page number.
When the recipe changes, Jetty reads the latest version from the page.

### Check your understanding

1. Why should `start` check `@server` before starting Jetty?
2. What problem can happen if you start Jetty twice on the same port?
3. What does `:join? false` let you keep doing?
4. Why are we using `#'my-handler` instead of just `my-handler`?

---

## Part 6 — The `stop` and `restart` functions

```clojure
(defn stop []
  (if @server
    (do
      (.stop @server)
      (reset! server nil)
      :server-stopped)
    :server-not-running))
```

Plain English:

> If there is a running server, stop it, empty the box, and say it stopped. If there is no server, say it was not running.

```clojure
(defn restart []
  (stop)
  (start))
```

Plain English:

> Stop the server, then start it again.

### Check your understanding

1. Why should `stop` set the atom back to `nil`?
2. What should happen if you call `(stop)` twice?
3. What does `(restart)` really do?

---

## Part 7 — Start CIDER

Inside Emacs, open `src/sample/core.clj`.

Run:

```text
M-x cider-jack-in
```

CIDER starts a REPL connected to your project.

When the REPL is ready, load your file:

```text
C-c C-k
```

That loads the current buffer into the running REPL process.

### Check your understanding

1. Do you need to run `lein repl` manually when using `cider-jack-in`?
2. What does `C-c C-k` do?
3. Why must the file be loaded before the REPL knows about `start`, `stop`, and `my-handler`?

---

## Part 8 — Start the web server from the REPL

In the REPL:

```clojure
(start)
```

Expected result:

```clojure
:server-started
```

Now test from the terminal:

```bash
curl -i http://localhost:8080/
```

You should see something like:

```text
HTTP/1.1 200 OK
Content-Type: text/plain

Hello from the REPL.
URI: /
```

Try another URI:

```bash
curl -i http://localhost:8080/pizza
```

Expected body:

```text
Hello from the REPL.
URI: /pizza
```

### Check your understanding

1. Who receives the browser or `curl` request first: your handler or Jetty?
2. Who calls your handler?
3. Why does `/pizza` appear in the response body?
4. What is the difference between calling `(my-handler {:uri "/pizza"})` and using `curl http://localhost:8080/pizza`?

---

## Part 9 — Change code without restarting Jetty

Change `my-handler`:

```clojure
(defn my-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "Changed without restarting Jetty.\n"
              "URI: " (:uri request) "\n")})
```

Now evaluate just that function.

In CIDER, put your cursor inside the `defn my-handler` form and use:

```text
C-M-x
```

Or reload the whole file:

```text
C-c C-k
```

Now run:

```bash
curl -i http://localhost:8080/pizza
```

Expected body:

```text
Changed without restarting Jetty.
URI: /pizza
```

You did not run `(restart)`.
You did not run `lein run`.
You changed the function in the running program.

That is the workflow.

### Check your understanding

1. Did Jetty restart when you changed the handler?
2. Why did Jetty still use the new handler code?
3. What CIDER command evaluates one top-level form?
4. What CIDER command reloads the whole buffer?

---

## Part 10 — When do you need to restart?

You usually do **not** need to restart Jetty when you change only handler logic.

You usually **do** need to restart when you change server setup, such as:

- port number
- Jetty options
- middleware wrapping done inside `start`
- anything that is captured when the server starts

Example:

```clojure
{:port 8081
 :join? false}
```

If you change the port from `8080` to `8081`, reload the file and then run:

```clojure
(restart)
```

### Check your understanding

1. If you change only the body string in `my-handler`, do you need `(restart)`?
2. If you change the port, do you need `(restart)`?
3. Why are these two cases different?

---

## Part 11 — Deliberate break/fix exercise

Do this only after the server works.

### Break 1: Wrong response key

Change this:

```clojure
:headers {"Content-Type" "text/plain"}
```

to this wrong version:

```clojure
:header {"Content-Type" "text/plain"}
```

Evaluate the handler and test with:

```bash
curl -i http://localhost:8080/
```

Question:

1. Does the body still appear?
2. Does the response header behave the way you expected?
3. Why is `:headers` the correct Ring key?

Fix it before continuing.

---

### Break 2: Old function copy

This exercise teaches why `#'my-handler` matters.

In `start`, temporarily change:

```clojure
(jetty/run-jetty #'my-handler
                 {:port 8080
                  :join? false})
```

to:

```clojure
(jetty/run-jetty my-handler
                 {:port 8080
                  :join? false})
```

Now reload the file and restart:

```clojure
(restart)
```

Change the text inside `my-handler`, evaluate only the handler, and refresh the browser or run `curl`.

Question:

1. Did the server immediately use the new handler?
2. What changed when you passed `my-handler` instead of `#'my-handler`?
3. Which one is better for this beginner REPL workflow?

After the experiment, put it back:

```clojure
#'my-handler
```

Reload and restart.

---

## Part 12 — Development loop you should memorize

For handler changes:

```text
edit handler
C-M-x or C-c C-k
refresh browser or run curl
observe result
```

For server setup changes:

```text
edit server setup
C-c C-k
(restart)
refresh browser or run curl
observe result
```

For direct function testing:

```text
call handler directly with a fake request map
observe the response map
then test through Jetty
```

---

## Part 13 — Quiz

Answer without looking back.

1. What is REPL-driven development?
2. Why is Clojure pleasant for this workflow?
3. What does `cider-jack-in` do?
4. What does `C-c C-k` do?
5. What does `C-M-x` do?
6. What does `:join? false` do?
7. Why do we store the Jetty server object in an atom?
8. Why do we use `defonce` for the server atom?
9. What does `#'my-handler` mean in this lesson?
10. When do you need to restart Jetty?
11. When can you avoid restarting Jetty?
12. Why is this not just a typing exercise?

---

## Exit ticket

Explain this without looking:

> I start Jetty once from the REPL. Jetty keeps running. When I change the handler and evaluate it, the running server can use the new handler because Jetty was given a live reference to the handler Var. This lets me develop by changing small pieces instead of restarting the whole app every time.

If you cannot explain that, you copied the workflow but did not learn it.
