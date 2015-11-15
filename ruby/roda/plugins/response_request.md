---
title: [Ruby, Roda - Web Framework, Plugins, ":response_request"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:response_request** plugin gives the response access to the related request instance via the `#request` method.

### Example:

{% highlight ruby %}
  plugin :response_request
{% endhighlight %}


---

### InstanceMethods


Set the response's request to the current request.

{% highlight ruby %}
  def initialize(env)
    super
    @_response.request = @_request
  end
{% endhighlight %}

--- 

### ResponseMethods

#### `#.request`

The request related to this response.  `attr_accessor :request`

