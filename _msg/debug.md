---
layout: page
title: Debug Page
#permalink: /about/
sitemap: false
published: false
---

## Content filtering

{% raw %}`{{ page.date }}`{% endraw %}

{% capture form %}
{% raw %}
<form
  action="https://formspree.io/f/xdordkpb"
  method="POST" markdown="0"
>
  <label>
    Your email: 
    <input type="email" name="email">
  </label>
  <label>
    Your message:
    &lt;textarea name="message"></textarea>
  </label>
  <!-- your other form fields go here -->
  <button type="submit">Send</button>
</form>
{% endraw %}
{% endcapture %}

{{ form | markdownif }}



{% comment %}
{% capture raw_html_content %}
{% include ar-contact-form.txt %}
{% endcapture %}

{{ raw_html_content | raw }}
{% endcomment %}
