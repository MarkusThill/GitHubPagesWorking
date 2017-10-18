---
layout: post
title: Using Slideshows in Jekyll
modified:
categories:
description:
tags: []
image:
  feature:
  credit:
  creditlink:
comments:
share:
date: 2017-10-18T14:45:46+02:00
---

When you have many images for a blog post it might make sense to organize all images in a slide-show. I found a sort of "plugin" that seems to deliver nice results. However, it was a little tricky to make it work. You can download all necessary files for the [jekyll-slideshow](https://github.com/lexoyo/jekyll-slideshow) from [https://github.com/lexoyo/jekyll-slideshow](https://github.com/lexoyo/jekyll-slideshow). After downloading, copy the folders (or the files within, if you already have these folders in your working directory) `lightslider`, `css`, `_slides` into your working directory. The folder `_slides` already contains two examples, which you can use initially for testing. Then you should copy the file `iframe.html` from the folder `_layouts` into your corresponding folder. Furthermore, from the file `_includes/head.html`  copy the line

{% highlight html %}
<script src="//ajax.googleapis.com/ajax/libs/jquery/1.11.0/jquery.min.js"></script>
 {% endhighlight %}

 into your own `head.html` and from your main config-file `_config.yml` copy the lines

{% highlight html %}
 defaults:
   -
     scope:
       path: "_slides"  #__
     values:
       layout: "iframe"

 collections:
   slides:
     output: true
{% endhighlight %}

into your config file. In order to resize the slideshows properly, you can add the following lines to `_includes/head.html`, so that you do not have to specify the height of the slideshow later:

{% highlight html %}
<script>
  function resizeIframe(obj) {
    obj.style.height = obj.contentWindow.document.body.scrollHeight + 'px';
  }
</script>
{% endhighlight %}

###
permalink: /slides/mypics1.html


{% highlight html %}
<iframe class="slideshow-iframe" src="{{ site.url}}/slides/my-pics1.html"
style="width:100%" frameborder="0" scrolling="no" onload="resizeIframe(this)"></iframe>`
{% endhighlight %}

<iframe class="slideshow-iframe" src="{{ site.url}}/slides/my-pics1.html" style="width:100%" frameborder="0" scrolling="no" onload="resizeIframe(this)"></iframe>

js to adjust height
