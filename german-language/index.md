---
layout: default
title: "German Language"
---
# German Language Blogs

Here are all my blogs about learning German:

<ul>
  {% for post in site.posts %}
    {% if post.url contains '/german-language/' %}
      <li><a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%B %d, %Y" }}</li>
    {% endif %}
  {% endfor %}
</ul>
