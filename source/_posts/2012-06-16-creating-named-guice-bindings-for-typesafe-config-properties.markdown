---
layout: post
title: "Creating named Guice bindings for typesafe Config properties"
date: 2012-06-16 17:49
comments: false
categories: [scala, guice, typesafe]
author: Alex Moffat <alex.moffat@gmail.com>
---

Guice provides an [@Named annotation](http://code.google.com/p/google-guice/wiki/BindingAnnotations) so that you can distinguish multiple bindings which have the same type. A canonical example is string or numeric configuration values. For example, to inject a string value into a constructor you could use.

``` java
    @Inject
    public MyDB(@Named("jdbc.url") String jdbcUrl) {
```

This requires a binding to provide a value linked to the name. You can create this with the `bindConstant()` method from within a module, for example

``` java
    protected void configure() {
        bindConstant().annotatedWith(Names.named("jdbc.url")).to("http://localhost:431");
    }
```

In general though you'd like to read constants like this from a configuration of some sort. The guice [Names class](http://google-guice.googlecode.com/git/javadoc/com/google/inject/name/Names.html) provides a convenience method that will create Named bindings for all of the properties in a [Properties](http://docs.oracle.com/javase/6/docs/api/java/util/Properties.html) object. We're not using Properties for configuration values though, we're using [typesafe Config](https://github.com/typesafehub/config).

I wanted the same feature for typesafe Config so I wrote a class to create bindings for all the properties in a typesafe Config object so that they could be accessed with @Named annotations.

<!-- more -->

The comments at the top of the file explain how it all works. Just one important thing to note, you need to have the Named bindings available before they are used. In guice you don't have explicit control over the order in which values are bound so you'll need to use the ConfigBinder at the top of a configure method and place the module you use it in at the end of the list of modules you use to create your injector.

{% gist 2946244 %}