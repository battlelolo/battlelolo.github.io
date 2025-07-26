---
layout: default
title: All Posts
permalink: /posts/
---

# All Posts

<div class="posts-container">
  {% for post in site.posts %}
    <div class="post-item">
      <div class="post-content">
        <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
        {% if post.excerpt %}
          <p class="post-excerpt">{{ post.excerpt | strip_html | truncate: 120 }}</p>
        {% endif %}
        {% if post.categories.size > 0 %}
          <div class="post-categories">
            {% for category in post.categories %}
              <span class="category-tag">{{ category }}</span>
            {% endfor %}
          </div>
        {% endif %}
      </div>
      <div class="post-meta">
        <time class="post-date">{{ post.date | date: "%B %d, %Y" }}</time>
      </div>
    </div>
  {% endfor %}
</div>

{% if site.posts.size == 0 %}
  <div class="no-posts">
    <p>No posts yet. Stay tuned for exciting content! 🚀</p>
  </div>
{% endif %}