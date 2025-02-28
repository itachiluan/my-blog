---
title: Welcome to my blog
---

This is simply a test to see what the outcome will be.

# Posts
<ul>
  {% for post in site.posts %}
    <li>
      <a href="/my-blog/{{ post.url }}">{{ post.title }}</a>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>
