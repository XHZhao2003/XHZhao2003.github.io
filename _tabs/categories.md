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
    {% assign display_name = site.data.category_names[cat_name] | default: cat_name %}
    <li><a href="/categories/{{ cat_name }}/index.html">{{ display_name }}</a></li>
  {% endfor %}
</ul>