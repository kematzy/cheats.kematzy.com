---
title: [Ruby, Roda - Web Framework, Plugins, ":error_handler"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction

The **:error_handler** plugin adds an `error` handler to the routing, so that if routing the request 
raises an `error`, a nice error message page can be returned to the user.

You can provide the error handler as a block to the plugin:

{% highlight ruby %}
plugin :error_handler do |e|
  "Oh No!"
end
{% endhighlight %}


Or later via the `error` class method:

{% highlight ruby %}
plugin :error_handler

error do |e|
  "Oh No!"
end
{% endhighlight %}


In both cases, the `exception` instance is passed into the block, and the block can return the request body via a string.

If an `exception` is raised, a new response will be used, with the default status set to `500`, before 
executing the error handler. 

The error handler can change the response status if necessary, as well set headers and/or write to the body, 
just like a regular request.


### ClassMethods


#### '#.error(&block)``

Install the given block as the error handler, so that if routing the request raises an exception, 
the block will be called with the exception in the scope of the Roda instance.



### InstanceMethods

#### `#.call`

If an error occurs, set the response status to 500 and call the error handler.

