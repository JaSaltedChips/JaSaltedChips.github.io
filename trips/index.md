---
layout: default
title: "Trips"
---
# Travel Blogs

Here are all my travel blogs:

<ul>
  {% for post in site.posts %}
    {% if post.url contains '/trips/' %}
      <li><a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%B %d, %Y" }}</li>
    {% endif %}
  {% endfor %}
</ul>
