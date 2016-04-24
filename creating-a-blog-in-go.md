---
title: Creating a website/blog in Go, 10 easy steps.
slug: creating-a-blog-in-go
published: true
posted: 2016-04-21
updated: 2016-04-21
---
#### It's actually not that hard
All you need is [net/http](https://godoc.org/net/http), and Go's built in templating, [html/template](https://godoc.org/html/template), and a few other packages to make things nicer.

This website is a decent example. I had a entirely static website, but I wanted to be able to write posts in markdown.
There are a lot of static site generators that work just fine, like [Hugo](https://gohugo.io/), or [Jekyll](http://jekyllrb.com/), but I wanted to be able to play with Go an various things like React.
I can't do that with an entirely static website, and while I could probably get it working with one of the generators and some [Cross-Origin Resource Sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) headers, I decided I would just write my own.

<center>![relevant xkcd: standards](https://imgs.xkcd.com/comics/standards.png)</center>

Yes, I know. One more! But it's my website, so whatever.

This is a really simple website, the `main.go` file is about 40 lines long, and the web package is about 200 lines long, including a bunch of helpers that could have been their own package.

```go
package main

import (
    "log"
    "net/http"
    "os"
    "time"

    "bitbucket.org/flexd/website/web"
    "github.com/julienschmidt/httprouter"
)

func main() {
    router := httprouter.New()
    router.GET("/", web.Homepage)

    router.GET("/post/:slug/", web.ShowPost)

    router.ServeFiles("/static/*filepath", http.Dir("./static"))
    log.Println("listening on ", os.Getenv("PORT"))
    err := http.ListenAndServe(os.Getenv("PORT"), httpLogger(router))
    if err != nil {
        panic(err)
    }
}

// cleanly log all HTTP requests
func httpLogger(router http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, req *http.Request) {
        startTime := time.Now()
        router.ServeHTTP(w, req)
        finishTime := time.Now()
        elapsedTime := finishTime.Sub(startTime)
        log.Println("request:", req.RemoteAddr, req.Method, req.URL, elapsedTime)
    })
}
```

This is really a example of how elegant a web application in Go can be. I'm sure there is room for improvements, but reading this code it's fairly easy to comprehend what it does, provided you know what a `router` is.

I am using the [httprouter](https://github.com/julienschmidt/httprouter) package here, because I wanted to try it out, but the `net/http` mux is usually good enough for most things, you just need a bit more boilerplate code.

`httpLogger` is an example of HTTP Middleware in Go. It uses the log package to print a line per request, and outputs this per request>

`2016/04/21 10:09:23 request: 188.166.60.190:40264 GET /post/creating-a-blog-in-go/ 352.423µs`

And that is all it does. You could easily write middleware to handle authentication, or set a custom HTTP Header, or do whatever you wish.
[Alex Edwards](https://twitter.com/ajmedwards) has written an [excellent guide](http://www.alexedwards.net/blog/making-and-using-middleware) about this on his blog.

The `web` package holds the web handlers (for lack of a better name), there are only two. One renders the index page, and the other renders a single post.

```go
// Homepage renders the homepage
func Homepage(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
    data := map[string]interface{}{"Posts": posts}
    renderTemplate(w, "index.tmpl", data)
    return
}

// ShowPost renders a single post
func ShowPost(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
    pMutex.Lock()
    post, ok := posts[ps.ByName("slug")]
    pMutex.Unlock()
    if !ok {
        http.Error(w, "Not Found", http.StatusNotFound)
        return
    }
    data := map[string]interface{}{"Post": post}
    renderTemplate(w, "post.tmpl", data)
    return
}
```


The `renderTemplate` function is a handy helper from [Matt Silverlock (@elithrar)](https://elithrar.github.io)'s post about `html/template`. It is a good read, read it [here](https://elithrar.github.io/article/approximating-html-template-inheritance/).

I have only modified it slightly to add HTML Minification with [github.com/tdewolff/minify](https://github.com/tdewolff/minify)
```go
 func renderTemplate(w http.ResponseWriter, name string, data map[string]interface{}) {
    ... // Rest of function here
     // Set the header and write the buffer to the http.ResponseWriter
     w.Header().Set("Content-Type", "text/html; charset=utf-8")
     m := minify.New()
     m.AddFunc("text/html", html.Minify)
     minified := bufpool.Get()
     defer bufpool.Put(minified)
     err = m.Minify("text/html", minified, buf)
     if err != nil {
         http.Error(w, "error", http.StatusInternalServerError)
         return
     }
     minified.WriteTo(w)
}
```

That probably is not the best way to do it, but it works for now. I'm thinking about writing a Caddy extension to do the minifying there.

Matt also suggests reading [Jan Newmarch's html/template tutorial](http://jan.newmarch.name/golang/template/chapter-template.html), which covers the basics of `html/template`. Also a good read!

We can take a look at the templates. This is `post.tmpl`, the template that shows you a single post.
```go
{{ define "content" }}
    <div class="pure-g">
        <div class="content pure-u-2-5">
            <ul id="stream">
                {{ with .Post }}
                {{ template "postrow" . }}
                {{ end }}
            </ul>
        </div>
    </div>
{{ end }}
```

The templating uses [nested template definitions](https://godoc.org/text/template#hdr-Nested_template_definitions) as you can see by the `{{ template "postrow" . }}` line, and `{{ define "content" }}`.

This means that I can reuse a template across the index and the single post, since they look the same.

`postrow.tmpl` looks like this:

```go
{{ define "postrow" }}
<li>
    <article>
        / <a href="/post/{{.Slug}}"><strong>{{.Slug}}</strong></a><time>{{.PostedDate}}</time>
        <h1>{{.Title}}</h1>
        {{.Body}}
    </article>
</li>
{{ end }}
```

And that's it really! There is not much more to show. There is some magic that watches the `posts` folder and reloads the posts if there is any changes, but that is not yet working because of an unknown bug in [fsnotify](https://github.com/fsnotify/fsnotify).

I will publish the code once it's in it's final form. Maybe someone else can use it to create a site of their own, and it's not entirely a waste of time :-)
