
Here is the full step-by-step for **Leiningen + Emacs + CIDER (`cider-jack-in`)** for the exact code from **1 Jetty Server**.

---

## 1. Project and code setup (once)

From terminal:

`lein new app sample cd sample`

Edit `project.clj`:


```clojure
  :dependencies [[org.clojure/clojure "1.11.1"]
                 [ring "1.15.3"]
                 [ring/ring-devel "1.15.3"];; <----add this! ]
```

Edit `src/sample/core.clj` to:


```clojure
(ns req-res.core
  (:require [ring.adapter.jetty])
  (:gen-class))

(defn my-handler [req]
  {:status 200
   :header {"Content-Type" "text/html"}
   :body "Hello me"})


;; --------------ADDED FROM HERE-----------------------
;; --------------ADDED FROM HERE-----------------------
(defonce my-server (atom nil))

(defn start []
  (reset! my-server
          (ring.adapter.jetty/run-jetty  my-handler
                            {:port 8080 :join? false})))

(defn stop []
  (.stop @my-server))

(defn restart []
  (stop)
  (start))
;; --------------TO HERE-----------------------
;; --------------TO HERE-----------------------

(defn -main   [& args]   
  (ring.adapter.jetty/run-jetty my-handler {:port 8080}))
```

---

## 2. Start a CIDER REPL with `cider-jack-in` (`c-jack-in`)

Inside `core.clj` buffer:

1. `M-x cider-jack-in` (or `M-x c-jack-in`)
    
    - CIDER will detect `project.clj` and start a **Leiningen REPL** for you.
    - Wait until the REPL buffer appears and you see a `user=>` or similar prompt.

You do **not** run `lein repl` manually when using `cider-jack-in`. CIDER does it for you.

---

## 3. Load your namespace into the REPL

In `core.clj` buffer:

- `C-c C-k`    or (`M-x cider-load-buffer`)

This:

- Compiles the file
- Loads `sample.core` into the REPL
- Switches the REPL’s current namespace to `sample.core` (you’ll see `sample.core=>` at the prompt)

Now the REPL knows about:

- `my-handler`
- `start`
- `stop`
- `restart`
- `server`

---

## 4. Start Jetty from the REPL (not via `-main`)

In the REPL buffer:

`(start)`

Now:

- Jetty is running on port 8081
- The Jetty server instance is stored inside the `server` atom: `@server`

Open browser:  
`http://localhost:8081` → You should see:  
`Hello my-server`

At this point you have the **REPL-controlled web server** running.

---

## 5. Typical development loop (REPL-driven)

### 5.1 Edit code

Change `my-handler` in `core.clj`, e.g.:

```clojure
(defn my-handler [request]   
  {:status 200    
   :headers {"Content-Type" "text/plain"}    
   :body "Hello from CIDER"})
```

### 5.2 Reload the code into the running process

In `core.clj` buffer:

- `C-c C-k` again (`cider-load-buffer`)

This redefines `my-handler` in the running JVM.

Because you’re using `wrap-reload` with `#'my-handler`, every request will see the latest definition:

- No need to restart Jetty just to change handler logic.
- Just reload (`C-c C-k`) and
- go to REPL (C-c C-z), then type `(restart)`
- refresh the browser.

### 5.3 Check in the browser

Reload `http://localhost:8081`  
The response should now say: `Hello from CIDER`.

This is the REPL-driven loop:

1. Edit function
2. `C-c C-k` (load buffer)
3. restart the server (restart) 
4. Browser refresh

---

## 6. Evaluating smaller pieces when debugging

Besides `C-c C-k`, in Emacs+CIDER you have:

- `C-c C-e` – eval expression before point
- `C-M-x` – eval the **top-level form** at point (e.g. a single `defn`)
- `C-c C-j` – `cider-switch-to-repl-buffer`
- `C-c M-p` – insert last REPL input in REPL buffer

Example: you’re on `defn my-handler ...`:

- Hit `C-M-x` to re-eval just that function.
- Refresh browser. No server restart.

Use these constantly. Don’t re-run whole buffer if you’re tweaking one function.

---

## 7. Stopping and restarting the server

In REPL:

`(stop)`

If you broke `server` somehow and `stop` throws NPE, be defensive:

`(when @server   (.stop @server))`

To restart the current server cleanly:

`(restart)`

If you’ve changed how `start` works (different port, middleware, etc.):

1. `C-c C-k` to reload code
2. `(restart)` to apply new config

---


## 9. When to use `-main` vs REPL

- In REPL-driven development (what you’re doing now):  
    Ignore `-main`. Everything is controlled via REPL (`start`, `stop`, `restart`).

- For production / “run like an app”:  
    Use:
    
    `lein run`
    
    which calls `-main`, or build a JAR:
    
    `lein uberjar java -jar target/sample-0.1.0-SNAPSHOT-standalone.jar`
    

Don’t mix the two mental models.

you can remove `-main` for now.

---
