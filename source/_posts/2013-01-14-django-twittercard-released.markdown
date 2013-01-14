---
layout: post
title: "Django TwitterCard Released"
description: "Django TwitterCard simplifies the process of working with Twitter cards in ... Django"
date: 2013-01-14 08:42
comments: true
categories: django opensource
---

I recently pushed a [Django TwitterCard](http://jasonleveille.com/django-twittercard/) to GitHub.  The project simplifies the process of working with [Twitter cards](https://dev.twitter.com/docs/cards) in Django.

{% blockquote @twitterapi https://dev.twitter.com/docs/cards %}
Twitter cards make it possible for you to attach media experiences to Tweets that link to your content. Simply add a few lines of HTML to your webpages, and users who Tweet links to your content will have a "card" added to the Tweet that’s visible to all of their followers.
{% endblockquote %}

<!--more-->

## Demonstration

With a small bit of configuration, Django TwitterCard can handle the creation of the necessary meta-tags related to your card.  Here's a quick example:

```python settings.py
TWITTERCARD_CONFIG = {
    'SITE': '@jleveille',
    'CREATOR': '@jleveille'
}
```

{% gist 4530272 %}

The resulting meta-tags follow:

```html
<meta name="twitter:card" content="summary">
<meta name="twitter:site" content="@jleveille">
<meta name="twitter:creator" content="@jleveille">
<meta name="twitter:url" content="http://url.tld/sub/">
<meta name="twitter:title" content="Django TwitterCard Example"/>
<meta name="twitter:description" content="Django TwitterCard simplifies the process of working with Twitter cards in ... Django">
```

## Download / Documentation

* [https://github.com/leveille/django-twittercard](https://github.com/leveille/django-twittercard)
* [http://jasonleveille.com/django-twittercard/](http://jasonleveille.com/django-twittercard/)
* [http://davidwalsh.name/twitter-cards](http://davidwalsh.name/twitter-cards)

## Twitter Cards and the Open Graph

It's important to note the relationship between Twitter cards and the Open Graph:

{% blockquote @twitterapi https://dev.twitter.com/docs/cards#open-graph %}
If you're already using OpenGraph to describe data on your page, it’s easy to generate a Twitter card without duplicating your tags and data. When the Twitter card processor looks for tags on your page, it first checks for the Twitter property, and if not present, falls back to the supported Open Graph property
{% endblockquote %}

If you're already using the following tags in the Open Graph, they won't be necessary in your Twitter card tags:

1. `og:url`
2. `og:description`
3. `og:title`
4. `og:image`
5. `og:image:width`
6. `og:image:height`
