---
title: [Ruby, Roda - Web Framework, Plugins, ":padrino_render"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:padrino_render** plugin adds rendering support that is similar to Padrino's.  While not 
everything Padrino's rendering supports is supported by this plugin (yet), it currently handles enough 
to be a drop in replacement for some applications.

{% highlight ruby %}
plugin :padrino_render, views: 'path/2/views'
{% endhighlight %}

Most notably, this makes the `#.render` method default to using the layout, similar to how the `#.view` 
method works in the render plugin.  

If you want to call render and not use a layout, you can use the `layout: false` option:

{% highlight ruby %}
render('test')                 # layout


render('test', layout: false) # no layout
{% endhighlight %}

Note that this plugin loads the **:partials** plugin.


---

### InstanceMethods


#### `#.render(template, opts=OPTS)`

Call view with the given arguments, so that render uses a layout by default.
