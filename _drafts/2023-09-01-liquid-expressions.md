---
#layout: post-argon
title: Using Liquid expressions
date: 2023-09-01 12:00:00 +0100
author: Igor
categories: Internal
tags: [liquid, expressions, variables, data]
#permalink: /post1/
#slug: lorem-ipsum
excerpt_separator: <!--more-->
#redirect_from: [/lorem, /post/lorem]
#published: false
last_modified_at: 2023-09-11 12:00:00 +0100
---

This post contains various liquid expression.

This will display page title: {{ page.title }}

<!--more-->

## Using Liquid expressions

### Using variables

https://jekyllrb.com/docs/step-by-step/03-front-matter/

[Variables](https://jekyllrb.com/docs/variables/)

| Param | Value |
| - | - |
| Page title | {{ page.title }} |
| Site property github_username | {{ site.github_username }} |
| Jekyll env | {{ jekyll.environment }} |
| Page author | {{ page.author }} |
| Page id | {{ page.id }} |
| Link | [Link to some post]({% post_url 2023-09-04-diagrams-as-code %}) |
| Current markdown engine |  {{ site.markdown }} |
| To display liquid code | {% raw %}`{{ page.date }}`{% endraw %} |

{{ page.last-modified-date }}


## Listings

### Listing "authors" collections
<ul>
{% for author in site.authors %}
  <li><a href="{{ author.url }}">{{ author.name }}</a></li>
{% endfor %}
</ul>

### Listing posts by author of this page:
<ul>
{% assign filtered_posts = site.posts | where: 'author', page.author %}
{% for post in filtered_posts %}
  <li><a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>

### Listing all posts
<ul>
{% for item in site.posts %}
  <li><a href="{{ item.url }}">{{ item.title | escape }}</a></li>
{% endfor %}
</ul>

### Listing "real" pages
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

### Listing all pages
<ul>
{% for item in site.pages %}
  <li><a href="{{ item.url }}">{{ item.url }} ({{ item.title }})</a></li>
{% endfor %}
</ul>

### Listing items in custom collection "msg"
<ul>
{% for item in site.msg %}
  <li><a href="{{ item.url }}">{{ item.title }}</a></li>
{% endfor %}
</ul>


### List tags
{% for item in site.tags %}
  [{{ item[0] }}]
  <ul>
  {% for post in item[1] %}
  <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
  </ul>
{% endfor %}


### List categories
{% for item in site.categories %}
  [{{ item[0] }}]
  <ul>
  {% for post in item[1] %}
  <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
  </ul>
{% endfor %}


### List site collections
<ul>
{% for item in site.collections %}
  <li>{{ item.label }}</li>
{% endfor %}
</ul>

{% if jekyll.environment == "production" %}
  Runnning production
{% endif %}

{% comment %}
<pre id="jekyll-debug"></pre>
<script>
  var obj = JSON.parse(decodeURIComponent("{{ site | jsonify | uri_escape }}"));
  var prettyJson = JSON.stringify(obj, null, 4);  // Pretty-printed JSON (indented 4 spaces).
  document.getElementById("jekyll-debug").textContent = prettyJson;
</script>
{% endcomment %}

{% comment %}
{{  site.github.versions | inspect }}

{{  site.github | inspect }}
{% endcomment %}
