---
layout: post
title: "Using the noscript Element to Hide HTML Elements on Page Load"
date: 2012-03-29 22:09
comments: false
categories: [css, javascript, tricks]
author: Olivier Modica <omodica@vast.com>
---

Sometimes a simple use case leads to a neat technical trick, and this is exactly what this short post is about.

This specific use case was quite simple, based on the following requirements:

* Show a simple search experience page that allows the end user to pick a value from a long list of options using a select element (e.g. the make of a car) to perform a search.
* Make the page search engine friendly by providing link elements for the top option values instead of a select element when JavaScript is not available.
* Serve the same HTML content to all clients regardless of user agent ids or capabilities (no cloaking or semi-cloaking). 

The requirements are straightforward and can be implemented using the `noscript` element as follows:

<!-- more -->

``` html
Pick a make:

<select>
    <option value="audi">Audi</option>
    <option value="bmw">BMW</option>
    <option value="chevrolet">Chevrolet</option>
    ...
</select>

<noscript>
    <a href="...">Audi</a>
    <a href="...">BMW</a>
    <a href="...">Chevrolet>
    ...
</noscript>
```

This looks fine when JavaScript is enabled (only the select element is rendered), but this isn't very good when JavaScript is disabled because both the select and link elements are rendered.

The first idea was to initially hide the select element (with CSS) and then make it visible once the page was loaded (with JavaScript). I thought we could do a lot better than that and looked for a simple and efficient way to provide a better user experience in this case, and came up with the following solution: inject CSS to hide the select element using a `noscript` element within the `head` section on the document.

``` html
<head>
    <noscript>
        <style type="text/css">
            .js-content { display: none; }
        </style>   
    </noscript>
</head>

<body>
    ...
    Pick a make:
    
    <select class="js-content">
        <option value="audi">Audi</option>
        <option value="bmw">BMW</option>
        <option value="chevrolet">Chevrolet</option>
        ...
    </select>
    
    <noscript>
        <a href="...">Audi</a>
        <a href="...">BMW</a>
        <a href="...">Chevrolet>
        ...
    </noscript>
    ...
</body>
```

In addition to being very simple this has the advantage of being a pure CSS solution and works on all browsers that I could test on (including Internet Explorer 6). I also verified this was compliant with [HTML5](http://www.w3.org/TR/html5/the-noscript-element.html#the-noscript-element).

While I'm not making the argument that this was a very good use case to begin with I hope this can be useful a useful trick for others - certainly I'll add it to my own bad of tricks.