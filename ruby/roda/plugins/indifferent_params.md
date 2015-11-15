---
title: [Ruby, Roda - Web Framework, Plugins, ":indifferent_params"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:indifferent_params** plugin adds a `#.params` instance method which returns a copy of the 
request params hash that will automatically convert symbols to strings.


### Example

{% highlight ruby %}
plugin :indifferent_params

route do |r|
  
  params[:foo]
  
end
{% endhighlight %}


The *params* hash is initialised lazily, so you only pay the penalty of copying the request params if 
you call the `#.params` method.

Note that there is a [**rack-indifferent** gem](http://github.com/jeremyevans/rack-indifferent/) that 
automatically makes Rack use indifferent params. 

Using [**rack-indifferent** gem](http://github.com/jeremyevans/rack-indifferent/) is faster and has 
some other minor advantages over the **:indifferent_params** plugin, though it affects `Rack` itself 
instead of just the `Roda` app that you load the plugin into.



### InstanceMethods

#### `#.params`

A copy of the request params that will automatically convert symbols to strings.

