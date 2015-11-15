---
title: [Ruby, Roda - Web Framework, Plugins, AllVerbs]
layout: default
asset\_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction

The all_verbs plugin adds methods for http verbs other than `#.get` and `#.post`.  


The following verbs are added, assuming [Rack](https://github.com/rack/rack) handles them: 

* `delete`
* `head`
* `options`
* `link`
* `patch`
* `put`
* `trace`
* `unlink`

These methods operate just like Roda's default `#.get` and `#.post` methods, so using them without any arguments 
just checks for the request method, while using them with any arguments also checks that the arguments 
match the full path.

### Example

{% highlight ruby %}
plugin :all_verbs

route do |r|
  
  r.delete do
    # Handle DELETE request
  end
  
  r.head do
    # Handle HEAD request
  end

  r.link do
    # Handle LINK request
  end

  r.patch do
    # Handle PATCH request
  end
  
  r.put do
    # Handle PUT request
  end
  
  r.options do
    # Handle OPTIONS request
  end
  
  r.trace do
    # Handle TRACE
  end
  
  r.unlink do
    # Handle UNLINK request
  end
  
end
{% endhighlight %}

The verb methods are defined via meta-programming, so there isn't documentation for the individual methods created.
