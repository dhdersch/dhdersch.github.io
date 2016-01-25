---
layout: post
title:  "GoLang: When to use string pointers"
date:   2016-01-23 11:00:00
categories: golang
comments: true
uid: dd2
tags: 
  - go 
  - golang 
  - string 
  - "*string" 
  - pointer
---

A string in Go is a value. Thus, a string cannot be `nil`. 

{% highlight go %}

x := "I am a string!"

x = nil // Won't compile, strings can't be nil in Go

{% endhighlight %}

However, a pointer to a string (or `*string`) can be `nil`.

{% highlight go %}

var x *string

x = nil // Compiles! String pointers in GoLang can be nil

{% endhighlight %}

A good rule of thumb is to use normal strings unless you **need** `nil`. Normal strings are easier and safer to use in Go. Pointers require you to write more code because you need to check that a `*string` has a value before dereferencing.

{% highlight go %}

func UseString(s *string) error {
    if s == nil {
        temp := "" // *string cannot be initialized
        s = &temp // in one statement
    }
    value := *s // safe to dereference the *string
}

{% endhighlight %}

An empty string value `""` and `nil` are not the same thing. When programming, if you can't think of reason you would need `nil`, then you probably don't need it or want it.

### So when should I use a pointer to a string?

There may be times when you should use `*string`. For instance, you should probably use a `*string` for struct properties when deserializing json or yaml (or anything) into a structure.

Consider code where a json document is deserialized into a struct consisting of normal string properties.

{% highlight go %}

package main

import (
    "encoding/json"
    "fmt"
)

type Config struct {
    Environment string
    Version     string
    HostName    string

}

func (c *Config) String() string {
    return fmt.Sprintf("Environment: '%v'\nVersion:'%v'\nHostName: '%v'", 
    c.Environment, c.Version, c.HostName)
}

func main() {

    jsonDoc :=`
        {
            "Environment" : "Dev",
            "Version" : ""
        }`

    conf := &Config{}
    json.Unmarshal([]byte(jsonDoc), conf)
    fmt.Println(conf) // Prints 
                      //   Environment: 'Dev'
                      //   Version:''
                      //   HostName: ''

}


{% endhighlight %}

You'll notice that both `Version` and `HostName` are stored as empty strings. This is correct behavior for `Version`, but should `HostName` really be an empty string?

The answer to that question depends on your program. If it is acceptable for a missing property to be unmarshalled into an empty string, then there is no problem. In other words, if you will handle missing json properties and empty json properties the same, then use a normal string.

But what if `""` is a valid configuration value for `HostName`, but a missing property is not valid?

Answer: use a `*string`.

{% highlight go %}

package main

import (
    "encoding/json"
    "fmt"
)

type ConfigWithPointers struct {
    Environment *string // pointer to string
    Version     *string
    HostName    *string
}

func (c *ConfigWithPointers) String() string {
    var envOut, verOut, hostOut string
    envOut = "<nil>"
    verOut = "<nil>"
    hostOut = "<nil>"

    if c.Environment != nil { // Check for nil!
        envOut = *c.Environment
    }

    if c.Version != nil {
        verOut = *c.Version
    }

    if c.HostName != nil {
        hostOut = *c.HostName
    }

    return fmt.Sprintf("Environment: '%v'\nVersion:'%v'\nHostName: '%v'",
        envOut, verOut, hostOut)
}

func main() {

    jsonDoc :=
        `
        {
            "environment" : "asdf",
            "hostName" : ""
        }
        `

    conf := &ConfigWithPointers{}
    json.Unmarshal([]byte(jsonDoc), conf)
    fmt.Println(conf) // Prints the following:
                      // Environment: 'asdf'
                      // Version:'<nil>'
                      // HostName: ''

}

{% endhighlight %}

Notice that with `*string`, you can now differentiate between a missing property or `null` value from an empty string value in the json document.

*tl;dr; Only use a string pointer `*string` if you need `nil`. Otherwise, use a normal string*


