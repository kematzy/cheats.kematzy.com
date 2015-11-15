---
title: [Ruby, Roda - Web Framework, Plugins, ":status_handler"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:status_handler** plugin adds a `#.status_handler` method which sets a block that is called 
whenever a response with the relevant response code with an empty body would be returned.

This plugin does not support providing the blocks with the plugin call; you must provide them to 
`status_handler` calls afterwards:

{% highlight ruby %}
  plugin :status_handler

  status_handler(403) do
    "You are forbidden from seeing that!"
  end
  
  status_handler(404) do
    "Where did it go?"
  end
{% endhighlight %}


Before a block is called, any existing headers on the response will be cleared.  So if you want to 
be sure the headers are set even in your block, you need to reset them in the block.


---

### ClassMethods


Install the given block as a status handler for the given HTTP response code.
#### `#.status_handler(code, &block)`


Freeze the hash of status handlers so that there can be no thread safety issues at runtime.
#### `#.freeze`

---

### InstanceMethods

If routing returns a response we have a handler for, call that handler.
def call
