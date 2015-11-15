---
title: [Ruby, Roda - Web Framework, Plugins, ":default_headers"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction

The **:default_headers** plugin accepts a hash of headers, and overrides the `#.default_headers` method 
in the response class to be a copy of the headers.

Note that when using this module, **you should NOT attempt to mutate of the values set in the default
headers hash**.

Example:

{% highlight ruby %}
plugin :default_headers, 'Content-Type': 'text/csv'
{% endhighlight %}


You can **modify the default headers later by loading the plugin again**:

{% highlight ruby %}
plugin :default_headers, 'Foo': 'bar'

plugin :default_headers, 'Bar': 'baz'
{% endhighlight %}



### ClassMethods

#### `#.default_headers`

The default response headers to use for the current class.



### ResponseMethods


#### `#.default_headers`

Get the default headers from the related roda class.


