---
layout: default
icon: fas fa-stream
order: 1
title: 分类
---

<h1>文章分类</h1>

<ul>
  {% assign categories = site.categories %}
  {% for category in categories %}
    {% assign cat_name = category[0] %}
    <li><a href="/category/{{ cat_name  }}/index.html">{{ cat_name }}</a></li>
  {% endfor %}
</ul>