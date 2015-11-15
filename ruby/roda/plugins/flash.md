---
title: [Ruby, Roda - Web Framework, Plugins, ":flash"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:flash** plugin adds a +flash+ instance method to `Roda`, for typical web application flash 
handling, where values set in the current flash hash are available in the next request.

With the example below, if a `POST` request is submitted, it will redirect and the resulting `GET` request 
will return `'b'`.

{% highlight ruby %}
plugin :flash

route do |r|

  r.is '' do
    
    r.get do
      flash['a']
    end

    r.post do
      flash['a'] = 'b'
      r.redirect('')
    end
    
  end
  
end
{% endhighlight %}



### InstanceMethods

#### `#.flash`

Access the flash hash for the current request, loading it from the session if it is not already loaded.


#### `#.call`

If the routing doesn't raise an error, rotate the flash hash in the session so the next request has access to it.
