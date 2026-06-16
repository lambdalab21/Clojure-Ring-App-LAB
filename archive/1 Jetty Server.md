
# Request

1. Make a project
```bash
   $ lein new app my-project-name
```

2. Add dependencies to the project
   Open project.clj file add `[ring "1.15.3"]` to the `dependencies`
   
   it should look like:
```clojure
  :dependencies [[org.clojure/clojure "1.11.1"]
                 [ring "1.15.3"]]
```
   
3. Retrieve dependencies files by
   ```bash
   $ lein deps
   ```
This downloads files to `~/.m2/repository`

3. Add `jetty` library `(:requre [ring.adapter.jetty :as jetty])` to the namespace:
   ```clojure
   (ns my-project-name.core)
    (:require [ring.adapter.jetty])
    (:gen-class))
   ```
   
4. Write code
```clojure
(defn -main
  [& args]
  (ring.adapter.jetty/run-jetty
    (fn [req] {:status 200
              :header {"Content-Type" "text/html"}
              :body "my response."})
    {:port 8080}))
```


5. Run the application
```bash
   $ lein run
```
   
6. Request `http://localhost:8080`

	a. curl
``` shell
$ curl http://localhost:8080
```
	
	b. httpie
``` shell
$ http http://localhost:8080
```
	
	c. browser
, or open a browser and go to the site above.

7. Refactor the **handler**.   A handler is a function that receives a map representing *HTTP Request*  and return a map representing *HTTP Response*:
```clojure
(defn my-handler [request]
  {:status 200
   :headers {"content-type" "text/plain"}
   :body   "Do you see me?"})
   
(defn -main
  [& args]
  (ring.adapter.jetty/run-jetty my-handler {:port 8080}))
```

## Killing the Server 
if you get an error like :
req-res.core> (start)
Execution error (BindException) at sun.nio.ch.Net/bind0 (Net.java:-2).
Address already in use
![[Pasted image 20260103074714.png]]

Follow this step:
1. Find the process with **list open files** command, `lsof`:
```Bash
sudo lsof -i :<PORT>
```
![[Pasted image 20260103074828.png]]

2. Kill the process
   ```Bash
   kill <PID>
   ```
In this example `kill 739681`

3.  if the process does not terminate use -9 flag.
```Bash
kill -9 <PID>
```


## Details

The function `ring.adapter.jetty/run-jetty` is used to start an embedded Jetty web server from a Clojure program, using the Ring web library. It serves a Ring handler according to a variety of options. This is a common method for developing and deploying Clojure web applications. 

Function signature

`(run-jetty handler options` takes two arguments: 

- **handler**: A Ring handler function that processes incoming requests and return a response.
- **options**: A map of key-value pairs to configure the server. 

Here is a basic example of how to use `run-jetty`.

``` clojure
(require '[ring.adapter.jetty :refer [run-jetty]])

(defn handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body "Hello, World!"})

(run-jetty handler {:port 3000})
```

- The `handler` function is a standard Ring handler that returns a map containing the response's status, headers, and body.
- `run-jetty` starts the server. The `:port` option specifies the port number.


How it works

The `ring.adapter.jetty` library serves as the glue between Ring and the embedded Jetty web server. 

- The `run-jetty` function first initializes an embedded Jetty web server instance.
- It then wraps the provided Ring `handler` in a way that allows Jetty to call it with a request map.
- Finally, it starts the Jetty server, which begins listening for HTTP requests on the configured port.


# Question
## What is an HTTP request?

An HTTP request is a message sent from a client (usually a web browser or program) to a server to ask for data or perform an action.

## What is an HTTP response?

An HTTP response is the message a server sends back after receiving and processing an HTTP request.

It has three main parts:

1. **Status line** – tells the result of the request.  
2. **Headers** – provide metadata about the response.  
3. **Body** – contains the actual data (optional).  

Browsers or clients use this information to display a webpage, handle an API response, or show an error depending on the status code.

## What is a handler?

An HTTP handler is the function or component in a web server that processes an HTTP request and generates a corresponding HTTP response.

In concrete terms:

- The **web server** (like Jetty, Express, or Flask) receives a request.
    
- It passes the request object to a **handler**.

## What is a server?
A server is a computer or software that listens for requests from clients and sends back responses.

It can serve many kinds of data:

- **Web servers** send HTML, CSS, JSON, etc. (e.g., Nginx, Apache, Jetty).
- **File servers** share files.
- **Database servers** store and return records.

For HTTP, the process is:

1. The server **listens** on a port (e.g., 80 or 8080).
2. A client (like a browser) sends an HTTP request.
3. The server runs a handler or route that generates a response.
4. The response goes back to the client.

## What is a handler?

A handler is a function that receives an input, processes it, and returns an output.
In web development, an **HTTP handler** specifically takes a _request_ and produces a _response_.

Example in **Clojure (Ring)**:

`(defn handler [request]   
{:status 200    :headers {"Content-Type" "text/plain"}    :body "Hello"})`

- `request` is a map describing what the client sent.
- The return value is a map describing what to send back.

Handlers are the core logic of servers—they decide what to do with incoming requests and what data to return.


# Reference
[source code](https://github.com/ring-clojure/ring/blob/master/ring-jetty-adapter/src/ring/adapter/jetty.clj)
[ring-clojure](https://ring-clojure.github.io/ring/ring.adapter.jetty.html)



