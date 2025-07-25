---
layout: default
title: All Posts
permalink: /posts/
---

# All Posts

<div class="posts-list">
  {% for post in site.posts %}
    <article class="post-preview">
      <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
      <p class="post-meta">{{ post.date | date: "%B %d, %Y" }}</p>
      {% if post.categories.size > 0 %}
        <p class="post-categories">
          {% for category in post.categories %}
            <span class="category-tag">{{ category }}</span>
          {% endfor %}
        </p>
      {% endif %}
      {% if post.excerpt %}
        <p class="post-excerpt">{{ post.excerpt | strip_html | truncate: 150 }}</p>
      {% endif %}
      <a href="{{ post.url }}" class="read-more">Read more →</a>
    </article>
  {% endfor %}
</div>

{% if site.posts.size == 0 %}
  <p>No posts yet. Stay tuned!</p>
{% endif %}