---
layout: post
title:  "Case-sensitive HTTP Headers with Go's net/http"
date:   2016-01-23 11:00:00
categories: golang
comments: true
ads: false
uid: dd2
tags: 
  - go 
  - golang 
  - aws 
  - "API Gateway" 
  - http
---

Today, Amazon Web Services released a number of updates to their services. 
In particular, they added a feature to [API Gateway](https://aws.amazon.com/api-gateway/) 
called [Usage Plans](https://aws.amazon.com/blogs/aws/new-usage-plans-for-amazon-api-gateway/).
Unfortunately, it seems that they've introduced a change that is not backwards-compatible with previous version of the service.

API Gateway allows the use of api key authentication. The keys are to be passed in using an HTTP header named `x-api-key`. 
Previously, the casing in this header didn't matter. `x-api-key`, `X-Api-Key`, and `X-API-KEY` were all treated the same.
However, after the recent changes, API Gateway seems to only accept `x-api-key`.

This is especially frustrating for Go's `http.Client` in the standard libary `net/http`.
The following code snippet can be used to add a header to a request:


{% highlight go %}

client := &http.Client{}
request, _ := http.NewRequest("GET", "https://someapi/someresource", nil)
request.Header.Set("x-api-key", "somelongapikey2349208759283")

{% endhighlight %}

However, `request.Header.Set(...)` ends up calling [CanonicalMIMEHeaderKey](https://github.com/golang/go/blob/master/src/net/textproto/reader.go#L554) 
on the header key, in this case `"x-api-key"`.
This converts `"x-api-key"` to `"X-Api-Key"`.

Note that `request.Header.Add("x-api-key", "somelongapikey2349208759283")` does the same thing.

So how can we set a header to be all lowercase? It turns out that `request.Header` is an alias of the type `map[string][]string`.
Thus, we can set it as lowercase with the following code:

{% highlight go %}

client := &http.Client{}
request, _ := http.NewRequest("GET", "https://someapi/someresource", nil)
r.Header["x-api-key"] = []string{ "somelongapikey2349208759283" }

{% endhighlight %}


*tl;dr; AWS released a backwards-incompatible change to their API. The Go Standard Library converts headers to canonical format.*


