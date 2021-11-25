# Page Toc Plugin

The **Page Toc** Plugin is for [Grav CMS](http://github.com/getgrav/grav) that generates a table of contents from a page's HTML header tags.

![](assets/page-toc.png)

## Installation

Installing the Page Toc plugin can be done in one of two ways. The GPM (Grav Package Manager) installation method enables you to quickly and easily install the plugin with a simple terminal command, while the manual method enables you to do so via a zip file.

### GPM Installation (Preferred)

The simplest way to install this plugin is via the [Grav Package Manager (GPM)](http://learn.getgrav.org/advanced/grav-gpm) through your system's terminal (also called the command line).  From the root of your Grav install type:

    bin/gpm install page-toc

This will install the Page Toc plugin into your `/user/plugins` directory within Grav. Its files can be found under `/your/site/grav/user/plugins/page-toc`.

### Manual Installation

To install this plugin, just download the zip version of this repository and unzip it under `/your/site/grav/user/plugins`. Then, rename the folder to `page-toc`. You can find these files on [GitHub](https://github.com/team-grav/grav-plugin-page-toc) or via [GetGrav.org](http://getgrav.org/downloads/plugins#extras).

You should now have all the plugin files under

    /your/site/grav/user/plugins/page-toc
	
## Configuration

Before configuring this plugin, you should copy the `user/plugins/page-toc/page-toc.yaml` to `user/config/plugins/page-toc.yaml` and only edit that copy.

Here is the default configuration and an explanation of available options:

```yaml
enabled: true     # Plugin enabled
active: true      # TOC processed by default for all page
start: 1          # Start header tag level (1 = h1)
depth: 6          # Depth from start (2 = 2 levels deep)
```

For example if you had a start of `3` and a depth of `3` you would get a TOC for `h3`, `h4`, and `h5`.

By default, The plugin is 'active' and will add header id attributes anchors for each header level found in a page.  You can set `active: false` and then activate on a page basis by adding this to the page frontmatter:

```yaml
page-toc:
  active: true
```

You can also configure which header tags to start and depth on when building the id attribute anchors by changing the `start` and `depth` values. This can also be done on a per-page basis.

## Usage

### Shortcode-like syntax in your content

You can use the following shortcode-like syntax in your content:

```md
[TOC] or [TOC/] or [toc] or [toc /]
```

This will replace the shortcode syntax with the Table of Contents with the `components/page-toc.html.twig` Twig template. Either the default one included in the `page-toc` plugin or an overridden version from your theme.

NOTE: It's not required to set the TOC plugin `active` if you use the shortcode syntax in your content.  That is a good enough indication that you want the plugin to be active.

### Twig Templating

When the plugin is `active` it will add anchors to the header tags of the page content as configured. You can simply include the provided Twig template:

```twig
{% include 'components/page-toc.html.twig' %}
```

You can also add your **table of contents** HTML in your Twig template directly with the provided `toc()` Twig function:

For example:

```twig
{% if config.get('plugins.page-toc.active') or attribute(page.header, 'page-toc').active %}
<div class="page-toc">
    {% set table_of_contents = toc(page.content) %}
    {% if table_of_contents is not empty %}
    <h4>Table of Contents</h4>
    {{ table_of_contents|raw }}
    {% endif %}
</div>
{% endif %}
```

or via the `toc_items()` function which rather than returning HTML directly returns objects and you can manipulate the output as needed:

```twig
{% macro toc_loop(items) %}
    {% import _self as self %}
    {% for item in items %}
        {% set class = loop.first ? 'first' : loop.last ? 'last' : null %}
        <li {% if class %}class="{{ class }}"{% endif %}>
            <a href="{{ item.uri }}">{{ item.label }}</a>
            {% if item.children|length > 0 %}
                <ul>
                    {{ _self.toc_loop(item.children) }}
                </ul>
            {% endif %}
        </li>
    {% endfor %}
{% endmacro %}

{% if config.get('plugins.page-toc.active') or attribute(page.header, 'page-toc').active %}
<div class="page-toc">
    {% set table_of_contents = toc_items(page.content) %}
    {% if table_of_contents is not empty %}
        <h4>Table of Contents</h4>
        <ul class="page-toc">
            {{ _self.toc_loop(table_of_contents.children) }}
        </ul>
    {% endif %}
</div>
{% endif %}
```

### Limiting levels in output

As well as limiting the levels that the page TOC plugin will use in the table of contents, you can also limit the levels that are actually displayed. To do this you can pass an optional `start`, and `depth` value to the `toc()` Twig function:

```twig
{% set table_of_contents = toc(page.content, 3, 3) %}
```

This will only display `H3` , and **3** levels deeper (up to `H5`) in the TOC output.

## Credits

The majority of this plugin's functionality is provided by the [PHP TOC Generator](https://github.com/caseyamcl/toc) library by [Casey McLaughlin](https://github.com/caseyamcl). So Thanks for making this a trivial plugin for Grav!


