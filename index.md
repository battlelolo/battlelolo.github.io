---
layout: default
title: Home
---

# Hello! Welcome to my blog.

I created this space to write about things I'm interested in:

**🚀 Tech**: Building cool new things.  
**⛓️ Blockchain**: Learning about the decentralized world.  
**🌱 Life**: Sharing my stories and thoughts.

## Recent Posts

<ul>
 {% for post in site.posts limit:5 %}
   <li>
     <a href="{{ post.url }}">{{ post.title }}</a>
     <span class="post-date">{{ post.date | date: "%B %d, %Y" }}</span>
   </li>
 {% endfor %}
</ul>

[View All Posts →](/posts)

## Categories
- [Tech](/category/tech)
- [Blockchain](/category/blockchain)
- [Life](/category/life)