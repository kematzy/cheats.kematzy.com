---
title: [Ruby, Roda - Web Framework, Plugins, ":class_level_routing"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:class_level_routing** plugin adds routing methods at the class level, which can be used instead 
of or in addition to using the normal `#.route` method to start the routing tree.  

If a request is not matched by the normal routing tree, the class level routes will be tried.  

This offers a more Sinatra-like API, while still allowing you to use a routing tree inside individual actions.

Here's an example using the **:class_level_routing** plugin:

{% highlight ruby %}
class App < Roda
  plugin :class_level_routing

  # GET / request
  root do
    request.redirect "/hello"
  end

  # GET /hello/world request
  get "hello/world" do
    "Hello world!"
  end

  # /hello request
  is "hello" do
    # Set variable for both GET and POST requests
    @greeting = 'Hello'

    # GET /hello request
    request.get do
      "#{@greeting}!"
    end

    # POST /hello request
    request.post do
      puts "Someone said #{@greeting}!"
      request.redirect
    end
  end
end
{% endhighlight %}


When using the the **:class_level_routing** plugin with nested routes, you may also want to use the
delegate plugin to delegate certain instance methods to the request object, so you don't have
to continually use `#.request` in your routing blocks.

Note that class level routing is implemented via a simple array of routes, so routing performance
will degrade linearly as the number of routes increases.  For best performance, you should use
the normal `#.route` class method to define your routing tree.  


This plugin does make it simpler to add additional routes after the routing tree has already been defined, though.




### ClassMethods

Routing methods that will store class level routes.

* `:root`
* `:on`
* `:is`
* `:get`
* `:post`
* `:delete`
* `:head`
* `:options`
* `:link`
* `:patch`
* `:put`
* `:trace`
* `:unlink`



