---
layout: default
title: Tech
permalink: /category/tech/
---

# 🚀 Tech Posts

Building cool new things - coding tutorials, development insights, and tech explorations.

<div class="posts-list">
  {% for post in site.categories.tech %}
    <article class="post-preview">
      <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
      <p class="post-meta">{{ post.date | date: "%B %d, %Y" }}</p>
      {% if post.excerpt %}
        <p class="post-excerpt">{{ post.excerpt | strip_html | truncate: 150 }}</p>
      {% endif %}
      <a href="{{ post.url }}" class="read-more">Read more →</a>
    </article>
  {% endfor %}
</div>

{% if site.categories.tech.size == 0 %}
  <p>No tech posts yet. Stay tuned for exciting content about building cool new things!</p>
{% endif %}

[← Back to all posts](/posts)