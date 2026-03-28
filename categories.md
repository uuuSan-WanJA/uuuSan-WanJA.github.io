---
layout: page
title: Categories
permalink: /categories/
---

{% for category in site.categories %}
## {{ category[0] }}

<ul>
  {% for post in category[1] %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    <small>({{ post.date | date: "%Y-%m-%d" }})</small>
  </li>
  {% endfor %}
</ul>
{% endfor %}
