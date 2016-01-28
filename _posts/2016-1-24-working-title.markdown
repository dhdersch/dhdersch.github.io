---
layout: post
title:  "AWS: DynamoDB and empty strings"
date:   2016-01-23 11:00:00
categories: aws
comments: true
ads: true
uid: dd3
published: false
tags: 
  - go 
  - golang 
  - string 
  - "*string" 
  - pointer
  - aws
  - Amazon Web Services
  - DynamoDB
---
Amazon Web Services' DynamoDB does not allow empty strings to be put into a table as an attribute value. 

I have not yet been able to find out why their DynamoDB development team chose not to support empty strings. Perhaps they will add support in the future. For now, they have implemented a `null` Attribute type.

This lack of empty string support can especially cause headaches for Go developers, since [Go strings cannot be `nil`]({% post_url 2016-01-23-golang-when-to-use-string-pointers %}).

# Workarounds

If you absolutely need empty strings put into a DynamoDB table, there are a couple of workarounds.

#### _Substitute empty string with `null`_

First, determine if you need `null` strings in your table (in addition to empty strings). Empty string and `null` are not the same thing, but if you only require one of the two then `null` can act as a suitable replacement for an empty string. This is the most straightforward workaround, so I recommend using it if you can.

One downside to this method is that it may require the use of string pointers in Go if you use Amazon's provided [dynamodbattribute](http://docs.aws.amazon.com/sdk-for-go/api/service/dynamodb/dynamodbattribute.html), which is part of their SDK.

In the following example, I use the dynamodbattribute library to convert my struct into the correct format to send to DynamoDB.

{% highlight go %}

var x *string

x = nil // Compiles! String pointers in GoLang can be nil

{% endhighlight %}

If, however, you need both `null` and `""` in your DynamoDB table, then substituting `null` for `""` will not work.

#### _Encode empty string_

A workaround that allows both `null` and `""` to both be used is to use a special string that represents an empty string such as `" "` or `"EMPTY_STRING`. Some might call this an escaped string. 

One pitfall to this method is that you will no longer be able to use your special string. This may or may not be a problem in your scenario. If it is a problem, use a string that is extremely unlikely to be put into your database as anything besides your special empty string.

