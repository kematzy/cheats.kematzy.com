---
title: [Ruby, Roda - Web Framework, Plugins, ":not_found"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:not\_found** plugin adds a +not_found+ class method which sets a block that is called whenever 
a `404` response with an empty body would be returned.

The usual use case for this is the desire for nice error pages if the page is not found.

You can provide the block with the plugin call:


{% highlight ruby %}
plugin :not_found do
  'Where did it go?'
end
{% endhighlight %}


  
Or later via a separate call to `#.not_found`:


{% highlight ruby %}
plugin :not_found

not_found do
  'Where did it go?'
end
{% endhighlight %}


Before `#.not_found` is called, any existing headers on the response will be cleared.  

So if you want to be sure the headers are set even in a `#.not_found` block, you need to reset them in the
`#.not_found` block.


---

### ClassMethods


#### `#.not_found(&block)`

Install the given block as the not_found handler.


---

### InstanceMethods


#### `#.call`

If routing returns a `404` response with an empty body, call the `#.not_found` handler.



