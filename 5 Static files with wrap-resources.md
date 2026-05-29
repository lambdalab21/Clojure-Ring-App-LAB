Start from the code in **4 Workflow Refresh** note

`wrap-resource` is the one that actually serves `/css/style.css` from `resources/public/...`. That’s independent of reload/refresh.

# 1. **Your project directory on disk**

A typical layout should look like this:
``` file
project/
  src/
  resources/
    public/
      css/
        style.css        <------ CSS file here
      img/
        logo.png         <------ images here
  project.clj

```

The **public/** directory is the conventional root for static assets.


# 2. **Ring’s _public folder_ rule**

When you use:

```clojure
(ring.middleware.resource/wrap-resource handler "public")`
```

Ring maps URLs **directly to files** under `resources/public`.

Examples:

| URL requested    | Actual file on disk              |
| ---------------- | -------------------------------- |
| `/css/style.css` | `resources/public/css/style.css` |
| `/img/logo.png`  | `resources/public/img/logo.png`  |
| `/favicon.ico`   | `resources/public/favicon.ico`   |

If the file isn’t in that exact location, it won’t be found. No excuses.

---

# 3. **Add the middleware correctly**

If you don’t wrap your handler, nothing is served.

Use one of these:
## wrap-resource`

```clojure
(require '[ring.middleware.file :refer [wrap-file]])

(def app
  (-> handler
      (wrap-file "resources/public")))

```
# 4. **Common mistakes you should stop making**

1. **Putting files in `/public` but folder is not under `resources/`**  
    Ring won’t look there.
    
2. **Using wrong paths inside HTML**  
    You do NOT write `/resources/public/css/style.css`.  
    You write `/css/style.css`.
    
3. **Forgetting to restart Jetty** if you're not using `wrap-reload`.
    
4. **Expecting Ring to serve static files without middleware**.  
    It won't.
    

---

# 5. Quick sanity check

Run this in REPL:

`(slurp "resources/public/css/style.css")`

If it errors, you put the file in the wrong place.



## Entire code
```clojure

(ns workflow.core
  (:require
   [ring.adapter.jetty :as jetty]
   [ring.middleware.reload :refer [wrap-reload] ]
   [ring.middleware.refresh :refer [wrap-refresh]]
   [ring.middleware.resource :refer [wrap-resource]])
  (:gen-class))

(defn my-handler [request]
  {:status 200
   :headers {"Content-Type" "text/html"}
   :body "<html><head><title>Hello d</title>
         <link rel=\"stylesheet\" href=\"/css/style.css\"></head>
<body>
<h1>Workflow with reload + refresh + static files</h1>
<p>in REPL try <code>(slurp \"resources/public/css/style.css\")</code></p>
</body></html>"})

(defonce server (atom nil))

(defn start []
  (reset! server
          (jetty/run-jetty
            (wrap-reload
              (wrap-refresh
                (wrap-resource #'my-handler "public")))
            {:port 8082
             :join? false})))


(defn stop []
  (.stop @server))

(defn restart []  (stop)
  (start))

(defn -main [& args]
  (ring.adapter.jetty/run-jetty my-handler {:port 8082}))
```


## using Hiccup2

```clojure
(h/html
 [:html
  [:head
   [:link {:rel "stylesheet"
           :href "/css/style.css"}]]
  [:body
   "Hello"]])
