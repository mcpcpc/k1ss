---
layout: default
---

Welcome to the KISS WIKI. Documentation and information about KISS and Linux in 
general. Written by the Community.

## Wiki Index

{% for image in site.static_files %}{% if image.path contains 'wiki/' %}[{{ image.path }}]({{ site.baseurl }}{{ image.path }}){% endif %}<br>{% endfor %}
