
# Starter Code
```clojure

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
          (jetty/run-jetty 
           (-> #'my-handler
               wrap-reload   ; Reloads code on server
               wrap-refresh) ; Triggers refresh in browser
           {:port 8080 :join? false})))

(defn stop []
  (.stop @server))

(defn restart []  
  (stop)
  (start))
```


