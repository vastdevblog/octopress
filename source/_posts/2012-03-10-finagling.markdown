---
layout: post
title: "Finagling"
date: 2012-03-10 16:08
comments: false
categories: [scala, finagle]
author: Alex Moffat <alex.moffat@gmail.com>
---

I've been playing with [Finagle](http://twitter.github.com/finagle/), or "Finagle, from Twitter" as the site refers to it, to see whether it will work for me as a framework to support a simple http rpc/rest server. The Finagle documentation is good but I wanted to explore in a complete simple example how I could implement

* Catching execptions from underlying services and returning 500 response codes to clients.
* Returning a timeout response if the underlying service takes too long to respone.
* Separating common process, such as extracting values from headers, from more specific service handling.
* Mapping from urls to services.
* Executing service code on a separate thread from the request processing thread.

<!-- more-->

The two files below are what I came up with. I've added lots of comments to explain what's going on so they should read clearly from top to bottom.

The ExampleServer responds to two urls, http://localhost:8080/ok will return a 200 response with some json, http://localhost:8080/timeout will return a 500 response because the underlying service takes too long to respond.

{% gist 2022320 ExampleServer.scala %}

 The SimpleModel is the Java object that's converted to json for return to the client. 

{% gist 2022351 ExampleModel.java %}


