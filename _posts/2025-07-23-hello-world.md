---
layout: default
title: "hello world"
date: 2025-07-23
categories: [life]
tags: [jekyll, github-pages]
---

# hello world

my first post ⭐


```javascript
const hello = "world";
function greet() {
    console.log(`Hello, ${hello}!`);
}
```

---

## 📌 Navigation
- [← All Posts](/posts)
- [🏠 Home](/)
<!-- - [📧 Contact](/contact) -->

{% if page.previous %}
**Previous:** [{{ page.previous.title }}]({{ page.previous.url }})
{% endif %}

{% if page.next %}
**Next:** [{{ page.next.title }}]({{ page.next.url }})
{% endif %}