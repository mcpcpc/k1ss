---
layout: default
title: blog
---

KISS BLOG
=========

RSS Feed available here: <{{ site.kiss.web }}/blog/blog.xml>

{% for file in site.pages %}{% if file.path contains 'blog/' %}[{{ file.path | replace: ".md", "" }}]({{ site.kiss.web }}/{{ file.path }})<br>{% endif %}{% endfor %}
