---
title: [Ruby, Roda - Web Framework, Plugins, ":partials"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:partials** plugin adds a +partial+ method, which renders templates without the layout.

{% highlight ruby %}
plugin :partials, views: 'path/2/views'
{% endhighlight %}

Template files are prefixed with an underscore:

{% highlight ruby %}
partial('test')     # uses _test.erb

partial('dir/test') # uses dir/_test.erb
{% endhighlight %}

This is basically equivalent to:

{% highlight ruby %}
render('_test')

render('dir/_test')
{% endhighlight %}

Note that this plugin automatically loads the :render plugin.


---

### InstanceMethods


#### `#.partial(template, opts=OPTS)`

Renders the given template without a layout, but prefixes the template filename to use with an underscore.


