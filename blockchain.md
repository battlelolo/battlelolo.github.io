---
layout: default
title: Blockchain
permalink: /category/blockchain/
---

# ⛓️ Blockchain Posts

Learning about the decentralized world - crypto, Web3, DeFi, and blockchain technologies.

<div class="posts-list">
  {% for post in site.categories.blockchain %}
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

{% if site.categories.blockchain.size == 0 %}
  <p>No blockchain posts yet. Coming soon - insights from the decentralized world!</p>
{% endif %}

[← Back to all posts](/posts)