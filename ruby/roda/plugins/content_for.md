---
title: [Ruby, Roda - Web Framework, Plugins, ":content_for"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:content_for** plugin is designed to be used with the **:render** plugin, allowing you to store 
content inside one template, and retrieve that content inside a separate template.  

Most commonly, this is so view templates can set content for the layout template to display outside
of the normal content pane.

The `content_for` template probably **only works with erb templates**, and requires that you don't 
override the `:outvar` render option.  

In the template in which you want to store content, call `#.content_for` with a block:

{% highlight ruby %}
<% content_for :foo do %>
  Some content here.
<% end %>
{% endhighlight %}


In the template in which you want to retrieve content, call `#.content_for` without the block:

{% highlight ruby %}
<%= content_for :foo %>
{% endhighlight %}

### InstanceMethods

#### `#.content_for(key, &block)`

If called with a block, store content enclosed by block under the given key.  

If called without a block, retrieve stored content with the given key, or return nil if there
is no content stored with that key.

