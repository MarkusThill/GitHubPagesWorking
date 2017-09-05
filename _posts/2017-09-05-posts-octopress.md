---
layout: post
title: Creating new Posts in Jekyll with Octopress
modified:
categories: Template
description:
tags: []
image:
  feature:
  credit:
  creditlink:
comments:
share:
date: 2017-09-05T20:09:19+02:00
---

# Creating new Posts and Pages
New Posts can easily be created using the gem Octopress. Make sure it is installed. Otherwise install with:
{% highlight bash %}
$ gem install octopress
{% endhighlight %}
Then, the following command will generate a new post in the folder ``/_posts``:
{% highlight bash %}
$ octopress new post "Post Title"
{% endhighlight %}
In case the post should be created in a specified folder, use:
{% highlight bash %}
$ octopress new post "Post Title" --dir myDir
{% endhighlight %}
Furthermore, Octopress allows to create new pages with:
{% highlight bash %}
$ octopress new page new-page/
{% endhighlight %}

# Highlighting Code
Highlighting code with Rouge can be done as follows:
{% highlight jekyll %}
{% raw  %}
{% highlight bash %}
$ octopress new page new-page/
{% endhighlight %}
{% endraw %}
{% endhighlight %}
There are many supported languages. Check out [this website](https://github.com/jneen/rouge/wiki/List-of-supported-languages-and-lexers) for a list.
