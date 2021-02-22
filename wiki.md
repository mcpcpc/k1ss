---
layout: default
title: wiki
---

WIKI
====

Welcome to the KISS WIKI. Documentation and information about KISS and Linux in 
general. Written by the Community.

Usage
-----

The wiki article index was designed for viewing in a terminal or your preferred
markdown viewer. For example, one could use 'curl' to download any one of the
articles below and pipe the resulting output into a terminal pager program:

    curl {{ site.kiss.web }}/wiki/boot/efistub.md | less

To simplify the command above, one could create a shell script function and 
place it in their shell rc file (e.g. ~/.ashrc). For example:

    # added to one's ~/.ashrc file
    wiki() {
        url='{{ site.kiss.web }}/wiki'
        curl "$url/${1:-kiss/style-guide}.md" | less
    }

To use this example script and view the example article from above, one would
simply type the following in a shell:

    $ wiki boot/efistub

Contribute
----------

To add a new article or modify an existing one, create a new Pull Request in
the community repository: <{{ site.kiss.git }}/website>

Each new article will be reviewed by the KISS wiki index owner for compliance
with the following style requirements:

*   Syntax is pandoc markdown (strict). 
    See <https://pandoc.org/MANUAL.html#pandocs-markdown> for more information.
*   Character width does not exceed 80. 
    An exception may be made for strings where a CR-LF cannot be reasonably 
    placed or would impact readability (e.g. long URLs).
*   All date and time related strings formatted per ISO 8601.

Article Index
-------------

{% for file in site.pages %}{% if file.path contains 'wiki/' %}<{{ site.kiss.web }}/{{ file.path }}><br>{% endif %}{% endfor %}
