---
title: Rewriting the Gophers invite form in Go
slug: rewriting-the-gophers-invite-form-in-go
published: true
posted: 2016-05-09
updated: 2016-05-09
---

We had been using [slackin](https://github.com/rauchg/slackin) for a while now to invite people to the [Gophers Slack](https://gophers.slack.com), and while it had been working okay, it had not been optimal.

For those of you who have not seen slackin before, it looks something like this:

<center>
![gophersinvite slackin](https://s.flexd.net/website-uploads/rewriting-the-gophers-invite-form-in-go/gophersinvite.png)
</center>

If you want to join us on Slack, visit the [invite form](https://invite.slack.golangbridge.org) to get invited! 

At the time I started writing this post the invite page was hosted on the [Heroku](https://heroku.com) free tier, mostly because I was lazy and there was a nice shiny "Deploy to Heroku"-button in the slackin README.

For the amounts of traffic the website gets, this should not have been be a problem. But `slackin` is written in Node.js, and somehow it was regularily using 200% of the allowed RAM. I would get lots of emails warning me, and the app would get killed by the OOM-killer.
This was a problem for us, because whenever the form stops working, people can't get in, and we get emails from them and we end up inviting people manually. Which is exactly the reason why we wanted a invite form in the first place.

![emails](https://s.flexd.net/website-uploads/rewriting-the-gophers-invite-form-in-go/heroku-logentries-emails.png)

In his 2008 blog post 'Hardware is Cheap, Programmers are Expensive', Jeff Atwood said

> Given the rapid advance of Moore's Law, when does it make sense to throw hardware at a programming problem? As a general rule, I'd say almost always.
> -- <cite>[Jeff Atwood][1]</cite>
[1]:https://blog.codinghorror.com/hardware-is-cheap-programmers-are-expensive/

It's the typical IT answer. It's efficient enough, just buy more servers!
We could have upgraded to a beefier dyno with enough memory, but that would not have solved anything if the node application had a memory leak causing it to use more memory. And I am not a fan of Javascript at all.

Since I had wanted to rewrite everything in Go for a long time, I took this as an opportunity to do just that, and here we are!


> Most people miss Opportunity because it is dressed in overalls and looks like work.
> -- <cite>[Thomas Edison][2]</cite>
[2]:http://www.quotedb.com/quotes/1375

I spent some time earlier this year integrating [ReCaptcha](https://www.google.com/recaptcha/intro/index.html), so I was already semi-familiar with the slackin code base, and I had a good idea of what needed to be done, so I just started writing a backend in Go, figuring I'd steal the entire frontend and that bit would require minimal changes.
It is not a complicated website. All we need is a form, and something that parses that form, and something to validate the recaptcha, and make calls to the Slack API.

Go has everything we need. [net/http](https://godoc.org/net/http) is an excellent choice for making websites, and the built-in [html/template](https://godoc.org/html/template) will work fine for most projects, including this. [nlopes/slack](https://github.com/nlopes/slack) is an excellent Slack client and [go-recaptcha/recaptcha](https://github.com/go-recaptcha/recaptcha) handles verifying the Recaptcha.

This is a snippet taken from slackin that renders the invite form.
```javascript
... *snip*
dom('form',
      // channel selection when there are multiple
      channels && channels.length > 1 && dom('select.form-item name=channel',
        channels.map(channel => {
          return dom('option', { value: channel, text: channel });
        })
      ),
      dom('input.form-item name=email type=email placeholder=you@yourdomain.com '
        + (!iframe ? 'autofocus' : '')),
      dom('input.form-item name=firstname type=text placeholder="First name" '
        + (!iframe ? 'autofocus' : '')),
      dom('input.form-item name=lastname type=text placeholder="Last name" '
        + (!iframe ? 'autofocus' : '')),
      coc && dom('.coc',
        dom('label',
          dom('input type=checkbox name=coc value=1'),
          'I agree to the ',
          dom('a', { href: coc, target: '_blank', }, 'Code of Conduct'),
          '.'
        )
      ),
      recaptcha && dom('br'),
      recaptcha && dom(`div.g-recaptcha data-sitekey=${sitekey}`),
      dom('button.loading', 'Get my Invite')
    ),
... *snip*
```

As you can see below, the author of slackin chose to generate HTML using node, and not a templating language. I have no idea why he would do such a thing, because it does not look any cleaner at all.
But luckily we do not have to deal with that, since we can grab the generated HTML by visiting the website!
I have just stolen the generated HTML, some javascript client code, and the CSS styling to make it look identical.

We ended up with about 200 lines of Go code. To be fair, this does not replicate all of the features in slackin. I did not bother with the badges or websockets (for live user count updates), simply because we did not really use those.

I saw that we did get a lot of traffic to the websocket URL when I deployed this new codebase, so it is possible someone has a badge for our Slack visible somewhere that is not working.

Here are some graphs that show memory usage and response times of both versions. It's not entirely a fair comparison, since the Go version does not implement any of the websocket functionality, but I'm fairly sure we could still use less memory than Node does ;-)

First we can compare the memory usage between the Node and Go versions:
<center>
![memory usage node](https://s.flexd.net/website-uploads/rewriting-the-gophers-invite-form-in-go/memory_usage_node.png)
Node Memory Usage
</center>

<center>
![memory usage go](https://s.flexd.net/website-uploads/rewriting-the-gophers-invite-form-in-go/memory_usage_go.png)
Go Memory Usage
</center>

As you can see, the Go version uses about 200MB less RAM. It's a bit hard to see in the graph there without a scale.

Next we compare the response time:
<center>
![response time node](https://s.flexd.net/website-uploads/rewriting-the-gophers-invite-form-in-go/response_time_node.png)
Node Response Time
</center>

<center>
![response time go](https://s.flexd.net/website-uploads/rewriting-the-gophers-invite-form-in-go/response_time_go.png)
Go Response Time
</center>

Go uses less memory and is quicker to respond to requests.
Additionally, it makes the developer happier he does not have to deal with node.js. :-)

The Go version also uses the [expvar](https://godoc.org/expvar) package to give us nice [metrics](https://invite.slack.golangbridge.org/debug/vars) that can be graphed by anyone. Read about using the expvar package [here](http://blog.ralch.com/tutorial/golang-metrics-with-expvar/) and [here](https://www.datadoghq.com/blog/instrument-go-apps-expvar-datadog/)

I have mine pushed into a [InfluxDB](https://influxdata.com/time-series-platform/influxdb/) instance using [Telegraf](https://influxdata.com/time-series-platform/telegraf/).
Looking at the data in Grafana I can get nice graphs like this, for any timespan:
<center>
![influx invite attempts](https://s.flexd.net/website-uploads/rewriting-the-gophers-invite-form-in-go/influx_invite_attempts.png)
</center>

<center>
![influx missing parameters](https://s.flexd.net/website-uploads/rewriting-the-gophers-invite-form-in-go/influx_missing_parameters.png)
</center>

<center>
![influx requests](https://s.flexd.net/website-uploads/rewriting-the-gophers-invite-form-in-go/influx_requests.png)
</center>


The only big question that remains is why the Node app was using so much memory.
I gave it more memory and let it sit without any traffic for a few days, and it still managed to use up all the available memory.

When I have time I am going to figure out why, but that will come as an update post later.

<center>
![node memory usage without traffic](https://s.flexd.net/website-uploads/rewriting-the-gophers-invite-form-in-go/heroku_node_24h_memory.png)
</center>

The code is available on GitHub at [flexd/slackinviter](https://github.com/flexd/slackinviter)
