---
layout: page
title: Test Page
permalink: /test
#redirect_from: [/test/]
sitemap: false
published: false
#hidden: true # does not work?
exclude: true # does not worlk too  {% unless page.exclude %}
---

 {% include linkedin-badge.html size='medium' %}

Links:

- [Lorem](../_drafts/2023-03-31-test-post.md)  
- [Liquid](../_drafts/2023-09-01-liquid-expressions.md)
- [site.json link]({% link _msg/site.json %})
- [site.json]({{ '/msg/site.json' |  relative_url}})
- [versions.json]({{ '/msg/versions.json' |  relative_url}})

Some [emojis](https://emojipedia.org/)

⚠️ Warning  
☁️ Cloud  
ℹ️ Info  

Check shortcodes: :cloud:

## Check some vars

{%- if false -%}
{% raw %}
{% include list-params.html size='medium' weight='heavy' %}
{% endraw %}
{%- endif -%}

{%- if true -%}
<pre id="jekyll-debug"></pre>
<script>
  var obj = JSON.parse(decodeURIComponent("{{ site.github.versions | jsonify | uri_escape }}"));
  var prettyJson = JSON.stringify(obj, null, 4);  // Pretty-printed JSON (indented 4 spaces).
  document.getElementById("jekyll-debug").textContent = prettyJson;
</script>
{%- endif -%}