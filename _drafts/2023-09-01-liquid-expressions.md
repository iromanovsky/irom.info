---
layout: post-argon
title:  Using Liquid expressions
date:   2023-09-01 12:00:00 +0100
author: Igor
categories: Blog
tags: [meta data, blog]
#permalink: /post1/
#slug: lorem-ipsum
excerpt_separator: <!--more-->
#redirect_from: [/lorem, /post/lorem]
#published: false
---

This post contains arious liquid expression.

This will display page title: {{ page.title }}

<!--more-->

## Using Liquid expressions

### Using variables

https://jekyllrb.com/docs/step-by-step/03-front-matter/

This will display page title: {{ page.title }}

This will display site properties from _config.yml: {{ site.github_username }}

Environment: {{ jekyll.environment }}

Page author: {{ page.author }}

Page id: {{ page.id }}

[Link to some post]({% post_url 2023-09-04-diagrams-as-code %})

### Using more complex expressions

Listing Authors

<ul>
    {% for author in site.authors %}
      <li>
       <a href="{{ author.url }}">{{ author.name }}</a>
      </li>
    {% endfor %}
</ul>

Listing posts by author of this page:
<ul>
  {% assign filtered_posts = site.posts | where: 'author', page.author %}
  {% for post in filtered_posts %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


Listing all posts:
<ul>
  {% for item in site.posts %}
    <li><a href="{{ item.url }}">{{ item.title }}</a></li>
  {% endfor %}
</ul>

Listing real pages:

{% assign default_paths = site.pages | map: "path" %}
{% assign page_paths = site.header_pages | default: default_paths %}
{% if page_paths %}
<ul>
{% for path in page_paths %}
  {% assign my_page = site.pages | where: "path", path | first %}
  {% if my_page.title %}
  <li><a href="{{ my_page.url | relative_url }}">{{ my_page.url | relative_url }} ({{ my_page.title | escape }})</a></li>
  {% endif %}
{% endfor %}
</ul>
{% endif %}

Listing all pages:
<ul>
  {% for item in site.pages %}
       <li><a href="{{ item.url }}">{{ item.url }} ({{ item.title }})</a></li>
  {% endfor %}
</ul>

Listing items in custom collections:
<ul>
  {% for item in site.pages1 %}
    <li><a href="{{ item.url }}">{{ item.title }}</a></li>
  {% endfor %}
</ul>


List tags:

{% for item in site.tags %}
  [{{ item[0] }}]
  <ul>
    {% for post in item[1] %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}


List categories:

{% for item in site.categories %}
  [{{ item[0] }}]
  <ul>
    {% for post in item[1] %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}


List site collections:

<ul>
  {% for item in site.collections %}
    <li>{{ item.label }}</li>
  {% endfor %}
</ul>

{% if jekyll.environment == "production" %}
  Runnning production
{% endif %}
