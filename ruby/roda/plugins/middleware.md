---
title: [Ruby, Roda - Web Framework, Plugins, ":middleware"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:middleware** plugin allows the `Roda` app to be used as `Rack` middleware.

In the example below, requests to `/mid` will return `Mid` by the `Mid` middleware, and requests to 
`/app` will not be matched by the `Mid` middleware, so they will be forwarded to `App`.


{% highlight ruby %}
class Mid < Roda
  plugin :middleware

  route do |r|
    
    r.is "mid" do
      "Mid"
    end
    
  end
  
end

class App < Roda
  
  use Mid
  
  route do |r|
    
    r.is "app" do
      "App"
    end
    
  end
  
end

run App
{% endhighlight %}


It is possible to use the `Roda` app as a regular app even when using the **:middleware** plugin.


---

### ClassMethods


#### `#.new(app)`

Create a Forwarder instead of a new instance if a non-Hash is given.



#### `#.route(&block)`

Override the route block so that if no route matches, we throw so that the next middleware is called.



---

### RequestMethods

#### `#.forward_next`

Whether to forward the request to the next application.  Set only if this request is being performed for middleware.
attr_accessor :forward_next

