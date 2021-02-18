---
layout: default
---

Welcome to the KISS WIKI. Documentation and information about KISS and Linux in 
general. Written by the Community.

## Usage

The wiki article index was designed for viewing in a terminal or your preferred
markdown viewer. For example, one could use 'curl' to download any one of the
pages below and pipe the resulting output into 'less':

    curl https://mcpcpc.com/k1ss/wiki/boot/efistub.txt | less

To simplify the command above, one could simply create a shell script that has 
the same exact function as above and place it in their shell rc (e.g. ~/.ashrc)
file. For example:

    # added to one's ~/.ashrc file
    wiki() {
        url='https://mcpcpc.com/k1ss/wiki'
        curl "$url/${1:-kiss/style-guide}.txt" | less
    }

To use the example script and view the example article from above, one would
simply type the following in a shell:

    $ wiki boot/efistub

## Contribute

To add a new article or modify an existing one, create a new Pull Request in
the community repository: http://github.com/mcpcpc/k1ss

Each new article will be reviewed by the KISS wiki index owner for compliance
with the following style requirements:

* syntax is pandoc markdown (strict)
* character width does not exceed 80

## Article Index
{% for image in site.static_files %}{% if image.path contains 'wiki/' %}[{{ image.path }}]({{ site.baseurl }}{{ image.path }}){% endif %}<br>{% endfor %}
