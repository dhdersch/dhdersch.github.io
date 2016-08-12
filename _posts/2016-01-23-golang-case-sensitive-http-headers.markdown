---
layout: post
title:  "GoLang: Case-sensitive HTTP Headers with net/http"
date:   2016-01-23 11:00:00
categories: golang
comments: true
ads: false
uid: dd3
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
Unfortunately, it seems that they've introduced a change that is not backwards compatible with previous versions of the service.

API Gateway allows the use of api key authentication. The keys are to be passed in using an HTTP header named `x-api-key`. 
Previously, the casing in this header didn't matter. `x-api-key`, `X-Api-Key`, and `X-API-KEY` were all treated the same.
However, after the recent changes, API Gateway seems to only accept `x-api-key`.

This is especially frustrating for applications using Go's `http.Client` which is part of the Go standard libary `net/http`.
The following code snippet can be used to add the `x-api-key` header to a request:


{% highlight go %}

client := &http.Client{}
request, _ := http.NewRequest("GET", "https://someapi/someresource", nil)
request.Header.Set("x-api-key", "somelongapikey2349208759283")
response, _ := client.Do(request)

{% endhighlight %}

However, `request.Header.Set(...)` ends up calling [CanonicalMIMEHeaderKey](https://github.com/golang/go/blob/master/src/net/textproto/reader.go#L554) 
on the header key, in this case `"x-api-key"`.
This converts `"x-api-key"` to `"X-Api-Key"`. Note that `request.Header.Add("x-api-key", "somelongapikey2349208759283")` does the same thing. 
While this is good behavior on Go's part, we need this header to be all lowercase.

So how can we set a header to be all lowercase? It turns out that `request.Header` is an alias of the type `map[string][]string`.
Thus, we can set the header key as lowercase (or whatever we want) with the following code:

{% highlight go %}

client := &http.Client{}
request, _ := http.NewRequest("GET", "https://someapi/someresource", nil)
r.Header["x-api-key"] = []string{ "somelongapikey2349208759283" }
response, _ := client.Do(request)

{% endhighlight %}


**tl;dr;** AWS released a backwards-incompatible change to an API that requires a lowercase HTTP header. The Go Standard Library converts headers to canonical format.


**UPDATE:** *Reddit user [/u/mwholt](https://www.reddit.com/user/mwholt) pointed out that this might have to do with [HTTP/2 requiring lowercase header fields](http://httpwg.org/specs/rfc7540.html#HttpHeaders).*

**UPDATE 2:** *It appears that AWS has fixed the issue with API Gateway. [https://forums.aws.amazon.com/thread.jspa?threadID=237153&tstart=0](https://forums.aws.amazon.com/thread.jspa?threadID=237153&tstart=0)*