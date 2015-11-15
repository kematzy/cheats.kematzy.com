---
title: [Ruby, Roda - Web Framework, Plugins, ":default_status"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:default_status** plugin accepts a block which should return a response status integer. 

This `integer` will be used as the default response status (usually `200`) if the body has been
written to, and you have not explicitly set a response status.

The block given to the block is `instance_exec`-ed in the context of the response.


### Example:

{% highlight ruby %}
  # Use 201 default response status for all requests
  plugin :default_status do
    201
  end
{% endhighlight %}



---

### ResponseMethods


#### `#.default_status`

`instance_exec` the **:default_status** plugin block to get the response status.

