---
title: About the design
slug: about-the-design
published: true
posted: 2016-04-19
updated: 2016-04-19
---
So I'm playing with the design of this website, what do you think? Here is some CSS to show syntax highlighting:
```css
body { font-family: helvetica, arial, sans-serif; }
.content {
    margin: 0 auto;
}
.content pre {
    overflow: auto;
    padding: .66em;
    background: #f4f4f4;
}
```
And some Go to show that is cool too.. Though no Go-specific highlighting enabled yet.

4/5 times this works!
```go
// handlers.go
package handlers

// e.g. http.HandleFunc("/health-check", HealthCheckHandler)
func HealthCheckHandler(w http.ResponseWriter, r *http.Request) {
    // A very simple health check.
    w.WriteHeader(http.StatusOK)
    w.Header().Set("Content-Type", "application/json")

    // In the future we could report back on the status of our DB, or our cache 
    // (e.g. Redis) by performing a simple PING, and include them in the response.
    io.WriteString(w, `{"alive": true}`)
}
```
What happens if I add more CSS here?
```css
body { font-family: helvetica, arial, sans-serif; }
.content {
    margin: 0 auto;
}
.content pre {
    overflow: auto;
    padding: .66em;
    background: #f4f4f4;
}
```


And here is a table!

Name    | Age
--------|------
Bob     | 27
Alice   | 23

# Header 1
## Header 2
### Header 3
> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
> 
> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
> id sem consectetuer libero luctus adipiscing.


> This is the first level of quoting.
>
> > This is nested blockquote.
>
> Back to the first level.


## This is a header.

1.   This is the first list item.
2.   This is the second list item.

Here's some example code:

    return shell_exec("echo $input | $markdown_script");


*   Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
    Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
    viverra nec, fringilla in, laoreet vitae, risus.
*   Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
    Suspendisse id sem consectetuer libero luctus adipiscing.
