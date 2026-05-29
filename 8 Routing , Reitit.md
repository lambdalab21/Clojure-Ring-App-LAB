
## 1 Add Dependency

Open **project.clj** file and add `[metosin/reitit "0.9.2"]`

```clojure
  :dependencies [[org.clojure/clojure "1.11.1"]
                 [ring "1.15.3"]
                 [ring/ring-devel "1.15.3"]
                 [ring-refresh "0.2.0"]
                 [hiccup "2.0.0"]
                 [metosin/reitit "0.9.2"]]  ;<-- ADD THIS!!
```

## 2. Add required

Add require `[reitit.ring :as ring]`
```clojure
  (:require
   [reitit.ring :as ring]               ;<---- Add this!!!
   [ring.adapter.jetty :as jetty]
   [ring.middleware.reload :refer [wrap-reload]]
   [ring.middleware.refresh :refer [wrap-refresh]])
  (:gen-class))
```


## 3. Define Your Routes

In Reitit, routes are defined as a **recursive vector of vectors**. This data-driven approach allows for clear nesting and shared route data (like middleware). 

``` clojure
(def app
  (ring/ring-handler
   (ring/router
    [["/" {:get home-page-handler}]
     ["/hello" {:get hello-page-handler}]
     ["/bye" {:get bye-page-handler}]
     ["/ping" {:get (fn [_] {:status 200 :body "pong"})}]] ) ))
```

Note: `_`  is a convention used to "ignore"



## 4 Error Handling
Add error handling like 404, in case the user enters paths that does not exist.

```clojure
(def app
  (ring/ring-handler
   (ring/router
    [["/" {:get home-page-handler}]
     ["/hello" {:get hello-page-handler}]
     ["/bye" {:get bye-page-handler}]
     ["/ping" {:get (fn [_] {:status 200 :body "pong"})}]] ) 
   (ring/create-default-handler
    {:not-found (constantly {:status 404 :body "Not Found"})
     :method-not-allowed (constantly {:status 405 :body "Method Not Allowed"})})))
```

## Entire Code

```clojure
(ns workflow.core
  (:require
   [reitit.ring :as ring]               ;<---- Add this!!!
   [ring.adapter.jetty :as jetty]
   [ring.middleware.reload :refer [wrap-reload]]
   [ring.middleware.refresh :refer [wrap-refresh]]
   [clojure.pprint :as pp])
  (:gen-class))


(defn home-page-handler [_]     ; `_` means ignore the required parameter.  Usually we use`request`.
  {:status  200
   :headers { " Content-Type " " text/plain " }
   :body "HOME"})

(defn hello-page-handler [_]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body "Hello"})

(defn bye-page-handler [_]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body "bye"})

;; Add route
(def app
  (ring/ring-handler
   (ring/router
    [["/" {:get home-page-handler}]
     ["/hello" {:get hello-page-handler}]
     ["/bye" {:get bye-page-handler}]
     ["/ping" {:get (fn [_] {:status 200 :body "pong"})}]] ) ;; `_`  is a convention used to "ingnore"
   (ring/create-default-handler
    {:not-found (constantly {:status 404 :body "Not Found"})
     :method-not-allowed (constantly {:status 405 :body "Method Not Allowed"})})))



(defonce server (atom nil))

(defn start []
  (reset! server
          (jetty/run-jetty (wrap-refresh (wrap-reload #'app))
                           {:port 8082 :join? false})))

(defn stop []
  (.stop @server))

(defn restart [] (stop)
  (start))
  

```



## Refrences
[Google AI](https://www.google.com/search?q=How+do+I+add+default+to+this+reitit+code+so+that+404+is+displayed+when+the+path+is+not+found.&gs_lcrp=EgZjaHJvbWUyBggAEEUYOdIBCjMxMDQxajBqMTWoAgiwAgE&sourceid=chrome&ie=UTF-8&udm=50&fbs=AIIjpHzQki16q-8Z7j6aseYi2jA_MjxEbhLu3BzW0raQHz9UWPRpvxEcaDfSC7oaAGD_yHppWjqgllc6clrCGiwoxVj-32hktxPyw9aVjVwg0ROaTDiSWIOUkpARr_3nXY6NrHVFU4sp4e8FDcEAdkH4rT70EWiQBWYof3FBnboGR_B0gdoyB45u4bubtjPvCtma2lZ-uKmOlsHcPQDTryyiSC_dm6jCGPez4ccsGHB3zT8yH5BpYpU&ved=2ahUKEwjL1rL4jOuRAxUmrokEHV__LX0Q0NsOegQIAxAB&aep=10&ntc=1&mstk=AUtExfBNb4EtrWkuaGtpZiRtBkjBp-hsTkGhvWB6CUL4aSoNRI7QY3B6yfHj9wBOZzjEL5VCgoA4IVAnt6XuCeu_HSWR5prRb6tMQVoHuEYKGCeUMPp_jKFNgCMYORrr2kCWObkQQ_LdJji9X5HeIglhZQU8zaPXoCeY4gelu2Kh9uQ1Ltsk4AlCTPijKX2-I-5ob4PjTc821ta7moYxW7yrIFThsFMQSrTUNAnHje-EbSbioB1GOJSckguLAktC7DAoc5eg5_zaY5KsvyddoToGRcDA_coqZquq1YR9rUjFszB_EXwAWEuoXxAOaV4V_j3fwEDERJe5csDOiw&csuir=1&mtid=c85WadW7DPDcptQPru_EqAw)
