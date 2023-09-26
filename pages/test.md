---
layout: page
title: Test Page
permalink: /test
#redirect_from: [/test/]
sitemap: false
published: false
---


<script src="https://platform.linkedin.com/badges/js/profile.js" async defer type="text/javascript"></script>
<div class="badge-base LI-profile-badge" data-locale="en_US" data-size="large" data-theme="light" data-type="VERTICAL" data-vanity="iromanovsky" data-version="v1"  style="float: right; padding-left: 1ch"></div>
  

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
