---
layout: default
title: "Investments"
---

Here are all my blogs about investments

<ul>
  {% for post in site.posts %}
    {% if post.categories contains 'investments-stock-market' %}
      <li><a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%B %d, %Y" }}</li>
    {% endif %}
  {% endfor %}
</ul>