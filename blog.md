---
layout: default
title: blog
---

KISS BLOG
=========

RSS Feed available here: <{{ site.blog }}/blog.xml>

{% for file in site.static_files %}{% if file.path contains 'blog/' %}[{{ file.path }}]({{ site.kiss }}/{{ file.path }})<br>{% endif %}{% endfor %}
