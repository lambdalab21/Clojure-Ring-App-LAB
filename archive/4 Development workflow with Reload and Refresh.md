
## Add middleware

When a client sends a request to a server, it passes through a pipeline or chain of **middleware** functions before reaching the final route handler or application logic. Each middleware function can perform specific tasks, and then it can either: 

- Pass the request (and potentially modified data) to the next middleware in the sequence using a `next()` function or callback.
- Send a response back to the client and end the cycle prematurely (e.g., in case of an authentication failure). 

The order in which middleware functions are defined and executed is crucial, as each can build upon the work of previous ones.


### `ring.middleware.reload`

Wrap the handler with **`ring.middleware.reload`** middleware, so that you do not have to restart the server every time you change functions.

Add a dependency to project.clj file. To use **ring.middleware.reload**, you need **`ring/ring-devel`** library.

```clojure
  :dependencies [[org.clojure/clojure "1.11.1"]
                 [ring "1.15.3"]
                 [ring/ring-devel "1.15.3"]] ;; <----add this!
```


Add a reference to **`reload`**, and call it **`wrap-reload`** to core.clj file
```clojure
(ns sample.core   
  (:require [ring.adapter.jetty :as jetty]             
            [ring.middleware.reload :refer [wrap-reload]] )  ; <-- add this
  (:gen-class))  
```

Wrap the handler with reload middleware.

From
```clojure
(defn start []
  (reset! my-server
    (ring.adapter.jetty/run-jetty  my-handler
                            {:port 8080 :join? false})))
```
To
```clojure
(defn start []   
  (reset! my-server           
    (jetty/run-jetty (wrap-reload #'my-handler)    ;;  <-- add `wrap-reload` 
                     {:port 8081 :join? false})))  
```

Notice `my-handler` became `#'my-handler`, which is the **Var** of the my-handler variable. 

mi
1. go to to REPL and `(restart)`
2. refresh the browser.

#### Test
1. update the handler's `:body`, and save (C-x C-s)
2. Refresh the browser (F5)

### `ring-refresh library`

I don't have to (restart) the server every time I change the code, but it will be much nicer if the browser refreshes when I save the file like *live-server*.  Add another middleware, **ring.refresh**.  It refreshes the browser when you save your file.  This library injects the reload JavaScript to `<head>`.   

Update **project.clj** file with **`ring-refresh "0.2.0"`**

```clojure
  :dependencies [[org.clojure/clojure "1.11.1"]
                 [ring "1.15.3"]
                 [ring/ring-devel "1.15.3"]
                 [ring-refresh "0.2.0"] ;; <----add this! ]
```

Update core.clj file from.
```Clojure
(ns sample.core   
  (:require [ring.adapter.jetty :as jetty]             
            [ring.middleware.reload :refer [wrap-reload]] )   
  (:gen-class))  
        
(defn my-handler [request]   
  {:status 200    
   :headers {"content-type" "text/plain"}    
   :body "Hello"})  

(defonce server (atom nil))  

(defn start []   
  (reset! server           
    (jetty/run-jetty (wrap-reload #'my-handler)                            
                     {:port 8081 :join? false})))  
  
(defn stop []   
  (.stop @server))  
  
(defn restart []   
  (stop)   
  (start))  
```


add `[ring.middleware.refresh :refer [wrap-refresh]]` 

```Clojure
(ns workflow.core
  (:require
   [ring.adapter.jetty :as jetty]
   [ring.middleware.reload :refer [wrap-reload]]
   [ring.middleware.refresh :refer [wrap-refresh]]) ; <-- Add this
  (:gen-class))

(defn my-handler [request]
  {:status 200
   :headers {"Content-Type" "text/html"}  ;; <-- `plaintext` to `html`
   :body "<html><head></head><body>Hello</body></html>"});; <-- need `<head>`  

(defonce server (atom nil))

(defn start []
  (reset! server
          (jetty/run-jetty (wrap-refresh (wrap-reload #'my-handler)) ;<-- Add
          {:port 8081 :join? false})))

(defn stop []
  (.stop @server))

(defn restart []  
  (stop)
  (start))

```


Steps
   1. `M-x c-jack-in`
   2. `C-c C-k`     or (M-x `cider-load-buffer`)
   3. REPL> (start)
   4. Update body
   5. C-M-x  to evaluate the top-level form

## Thread Macro

To make the code easier to read use the **thread macro** syntactic sugar.

update this 
```clojure

(defn start []
  (reset! server
          (jetty/run-jetty (wrap-reload (wrap-refresh #'my-handler)) 
          {:port 8081 :join? false})))
```

to use **thread macro**
```clojure

(defn start []
  (reset! server
          (jetty/run-jetty 
           (-> #'my-handler
               wrap-reload   ; Reloads code on server
               wrap-refresh) ; Triggers refresh in browser
           {:port 8081 :join? false})))
```


## Order of wrap
**`wrap-reload` should be outside `wrap-refresh`** in most dev setups.

Reason (practical, not theoretical): `wrap-refresh`’s job is to reload namespaces/files before the request hits your handler. If `wrap-reload` is outside, you get the expected dev behavior: code changes are picked up and the refreshed handler is what runs.

Use this order:
```clojure
(wrap-reload   
  (wrap-refresh))`
```

With the thread macro:
```clojure
(-> #'my-handler     
  (wrap-refresh)     
  (wrap-reload))
```

### Clarification so you don’t get confused

- If you swap `wrap-reload` and `wrap-refresh`, you’ll often still “see something work,” but you can get inconsistent behavior where one layer reloads while the other serves stale references. Don’t do that unless you know exactly why.

## Order of wrap (perplexity)

Use `wrap-refresh` _outside_ `wrap-reload`, i.e.:

```clojure
(wrap-refresh (wrap-reload #'handler))`
```

## Why this order

- `wrap-reload` should be the **inner** middleware so it can see and re-evaluate the freshest version of your handler var on each request.[practical+2](https://practical.li/clojure-web-services/projects/leiningen/todo-app/reloading-the-application/)​
    
- `wrap-refresh` operates on the resulting HTML response (injecting its JavaScript to auto-refresh the browser), so it should be **outermost** and see the final response after all other middlewares, including `wrap-reload`, have run.[clojars+1](https://clojars.org/ring-refresh)​
    

So the usual dev stack is:

```clojure
(def app   
  (-> #'handler      
    wrap-reload      
    wrap-refresh))`
```

which is equivalent to `(wrap-refresh (wrap-reload #'handler))`.[practical+1](https://practical.li/clojure-web-services/projects/leiningen/todo-app/reloading-the-application/)​
