---
layout: default
title: Life
permalink: /category/life/
---

# 🌱 Life Posts

Sharing my stories and thoughts - personal experiences, reflections, and life lessons.

<div class="posts-list">
  {% for post in site.categories.life %}
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

{% if site.categories.life.size == 0 %}
  <p>No life posts yet. Stay tuned for personal stories and thoughts!</p>
{% endif %}

[← Back to all posts](/posts)