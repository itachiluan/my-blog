---
title: Welcome to my blog
---

This is simply a test to see what the outcome will be.

## Posts
<ul class="post-list">
  {% for post in site.posts %}
    <li>
      <h3>
        <a class="post-link" href="/my-blog/{{ post.url }}">{{ post.title }}</a>
      </h3>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>
