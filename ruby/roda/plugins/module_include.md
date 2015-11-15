---
title: [Ruby, Roda - Web Framework, Plugins, ":module_include"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:module_include** plugin adds `#.request_module` and `#.response_module` class methods for adding 
modules/methods to request/response classes.  

It's designed to make it easier to add request/response methods for a given roda class.  

To add a module to the request or response class:

{% highlight ruby %}
Roda.request_module SomeRequestModule

Roda.response_module SomeResponseModule
{% endhighlight %}

Alternatively, you can pass a block to the methods and it will create a module automatically:

{% highlight ruby %}
Roda.request_module do

  def description
    "#{request_method} #{path_info}"
  end
  
end
{% endhighlight %}


---

### ClassMethods


#### `#.request_module(mod = nil, &block)`

Include the given module in the request class. If a block is provided instead of a module, create a 
module using the the block. 

**Example:**


{% highlight ruby %}

Roda.request_module SomeModule

Roda.request_module do
  
  def description
    "#{request_method} #{path_info}"
  end
  
end

Roda.route do |r|
  
  r.description
  
end
{% endhighlight %}



#### `#.response_module(mod = nil, &block)`

Include the given module in the response class. If a block is provided instead of a module, create a 
module using the the block. 

**Example:**

{% highlight ruby %}

Roda.response_module SomeModule

Roda.response_module do
  
  def error!
    self.status = 500
  end
  
end

Roda.route do |r|
  
  response.error!
  
end
{% endhighlight %}

