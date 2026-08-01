# 4G. Middleware Teacher Guide and Answer Key

## Oral exam questions

Ask these without letting the student look at the file.

1. What is the shape of a handler? Function: request map -> response map
2. What is the shape of middleware? Function: handler -> new handler
3. Why is the return value of middleware also a handler? Middleware returns a function that still accepts a request and returns a response. 
4. In `wrap-log-request`, who prints the URI? The middleware function. 
5. In `wrap-powered-by`, who creates the original response? The inner handler. 
6. In `wrap-powered-by`, who changes the response? Middleware. 
7. Does `wrap-add-student-name` mutate the request? No. It builds a new request map. 
8. In nested middleware, which wrapper sees the request first? The outermost wrapper. 
9. Which wrapper sees the response last? The outermost wrapper.  
10. What does the thread macro do? It rewrites nested calls into a readable left-to-right pipeline. 
11. Is the thread macro required for middleware? No. 
12. Why are libraries not needed to learn middleware? middleware is just ordinary functions that take and return handlers
