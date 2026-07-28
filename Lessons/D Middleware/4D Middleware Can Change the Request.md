# 4D. Middleware Can Change the Request

## 1. Add a request-changing middleware

Add this:

```clojure
(defn wrap-add-student-name [handler]
  (fn [request]
    (let [new-request (assoc request :student/name "Student")]
      (handler new-request))))
```
---

## 2. Change the handler to read the added value

Change `my-handler`:

```clojure
(defn my-handler [request]
  {:status 200
   :headers {"Content-Type" "text/plain"}
   :body (str "Hello from the REPL.\n"
              "URI: " (:uri request) "\n"
              "Student: " (:student/name request) "\n")})
```

Now the handler expects that something may have added `:student/name`.

---

## 3. Test directly

In the REPL:

```clojure
(def named-handler
  (wrap-add-student-name my-handler))
```

Call:

```clojure
(named-handler {:uri "/student"})
```

Expected body includes:

```text
Student: Student
```

Now call the plain handler:

```clojure
(my-handler {:uri "/student"})
```

Expected body includes:

```text
Student:
```

or possibly:

```text
Student: nil
```

### Stop and answer

1. Which function adds `:student/name`?
2. Which function reads `:student/name`?
3. Does the original request map get mutated?
4. What does `assoc` return?
5. Why is the new value visible to the handler?

---

## 4. Wrap the app

Use nested wrapping:

```clojure
(def app
  (wrap-powered-by
    (wrap-log-request
      (wrap-add-student-name
        #'my-handler))))
```

Read from the inside out when building:

```text
#'my-handler
↓
wrap-add-student-name
↓
wrap-log-request
↓
wrap-powered-by
```

But read from the outside in when the request arrives:

```text
request
↓
wrap-powered-by
↓
wrap-log-request
↓
wrap-add-student-name
↓
my-handler
```

---

## 5. Test through Jetty

Reload:

```text
C-c C-k
```

Restart:

```clojure
(restart)
```

Test:

```bash
curl -i http://localhost:8080/student-test
```

You should see the student name in the body.

---

## 6. Why this matters

Many real middlewares do this kind of work.

They may add information such as:

- parsed parameters
- session data
- authenticated user
- request ID
- development reload behavior

But do not think about those libraries yet.

The core idea is:

> Middleware can return a new request map before calling the handler.

---

## 7. Check your understanding

1. What does `wrap-add-student-name` receive?
2. What does it return?
3. Does it change the request or the response?
4. What function finally receives the changed request?
5. Why should we say “new request map” instead of “mutated request map”?
6. Why does this fit Clojure's style?

---

## 8. Exit ticket

Explain this without looking:

> A middleware can create a new request map, pass that new map to the handler, and let the handler behave as if that extra information was already part of the request.

If you cannot explain that, repeat this lesson.
