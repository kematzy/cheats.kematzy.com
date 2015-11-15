---
title: [Ruby, Roda - Web Framework, Plugins, ":h"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction

The **:h** plugin adds an `#.h` instance method that will HTML escape the input and return it.

The following example will return "&lt;foo&gt;" as the body.

{% highlight ruby %}
plugin :h

route do |r|

  h('<foo>')
  
end
{% endhighlight %}


### InstanceMethods

#### `#.h(s)`

HTML escape the input and return the escaped version.
