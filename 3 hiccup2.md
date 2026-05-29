
Start with "1 Jetty Server"

1. Add **hiccup2** library, `[hiccup "2.0.0"]`

```clojure
  :dependencies [[org.clojure/clojure "1.11.1"]
                 [ring "1.15.3"]
                 [hiccup "2.0.0"]]
```

  2. Retrieve dependencies files from the repository into `~/.m2/` directory by
   ```bash
   $ lein deps
   ```
   
3. load `hiccup2` library `(:requre [hiccup2.core])` to the namespace:
   ```clojure
   (ns my-project-name.core)
    (:requre [ring.adapter.jetty]
             [hiccup2.core :as h])
    (:gen-class))
   ```
   
   4. Replace `:body` of the handler with hiccup2, `(str (h/html [:html [:head] [:body]]))`
``` clojure
(defn hello-html []
  (h/html [:html
            [:head
              [:title "Hello"]]
            [:body 
              [:h1 "Hello! My HTML"]
              [:div 
                [:p "This is my paragraph."]]]]))

(defn my-handler [request]
  {:status 200
   :header {"content-type" "text/html"}
   :body   (str hello-html))
   
(defn -main
  [& args]
  (ring.adapter.jetty/run-jetty my-handler {:port 8080}))
```


3. Run the application
   ```bash
   $ lein run
   ```
   
4. Request `http://localhost:8080`

# Examples

Hiccup2 is a Clojure library for generating HTML using Clojure data structures. Here are some examples demonstrating its usage:

1. Basic HTML Structure:

``` clojure
(require '[hiccup2.core :as h])

(str (h/html [:html             
			 [:head [:title "My Page"]]             
			 [:body [:h1 "Welcome"]                    
			        [:p "This is a paragraph."]]]))
```

This example generates a complete HTML document with a head and body, including a title, heading, and paragraph.

2. Elements with Attributes:

``` clojure
(require '[hiccup2.core :as h])

(str (h/html [:div {:id "main-content" :class "container"}             
             [:p "Content goes here."]]))
```

This demonstrates adding attributes like `id` and `class` to an HTML element using a map.

3. CSS-like Shortcuts for ID and Class:

```clojure
(require '[hiccup2.core :as h])

(str (h/html [:div#header.nav-bar "My Header"]))
```

This example utilizes the convenient CSS-like syntax to define an element with both an ID and multiple classes.

4. Looping and Dynamic Content:

``` clojure
(require '[hiccup2.core :as h])

(str (h/html [:ul             
             (for [item ["Apple" "Banana" "Cherry"]]               
               [:li item])]))
```

This demonstrates how to generate dynamic lists using Clojure's `for` macro within Hiccup2.

5. Escaping and Raw Content:

```clojure
(require '[hiccup2.core :as h])

  (str (h/html [:p "Tags in HTML are written with " "<>"]))
   ;; Output: "<p>Tags in HTML are written with &lt;&gt;</p>"
  
  (str (h/html [:p (h/raw "<span>Unescaped content</span>")]))
   ;; Output: "<p><span>Unescaped content</span></p>"
```

Hiccup2 automatically escapes strings by default. The `h/raw` function can be used to insert unescaped HTML.

6. Composing Components:

```clojure
(require '[hiccup2.core :as h])

(defn my-button [label]  
  [:button {:type "button"} label])
  
(str (h/html [:div             
              (my-button "Click Me")             
              (my-button "Another Button")]))
```

This shows how to create reusable components as Clojure functions that return Hiccup2 data structures.

These examples illustrate the core features of Hiccup2, demonstrating how to construct various HTML elements and structures using Clojure's data literals.

# References

[Practicalli](https://practical.li/clojure-web-services/projects/leiningen/todo-app/hiccup/)
[Sysntax](https://weavejester.github.io/hiccup/syntax.html)
[Hiccup 2.0.0-RC1](https://weavejester.github.io/hiccup/)
[Github Repo](https://github.com/weavejester/hiccup)
