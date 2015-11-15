---
title: [Ruby, Roda - Web Framework, Plugins, ":symbol_views"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:symbol_views** plugin allows match blocks to return symbols, and consider those symbols as 
views to use for the response body.  


So you can take code like:

{% highlight ruby %}
r.root do
  view :index
end

r.is "foo" do
  view :foo
end
{% endhighlight %}


and DRY it up:


{% highlight ruby %}
  r.root do
    :index
  end

  r.is "foo" do
    :foo
  end

{% endhighlight %}


### TODO:  add more information and examples here.