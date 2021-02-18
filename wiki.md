---
layout: default
---

Welcome to the KISS WIKI. Documentation and information about KISS and Linux in 
general. Written by the Community.

## Usage

The wiki article index was designed for virwing in a terminal or your preferred markdown viewer.

## Contribute

In add a new article or modify an existing one, create a new Pull Request in the community repository:
http://github.com/mcpcpc/k1ss

Each new article will be reviewed for compliance with the following style requirements:

* syntax is pandoc markdown (strict)
* character width does not exceed 80

## Article Index
{% for image in site.static_files %}{% if image.path contains 'wiki/' %}[{{ image.path }}]({{ site.baseurl }}{{ image.path }}){% endif %}  {% endfor %}
