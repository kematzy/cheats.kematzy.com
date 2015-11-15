---
title: [Ruby, Roda - Web Framework, Plugins, ":halt"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:halt** plugin augments the standard request `#.halt` method to allow the response `status`, `body`, 
or `headers` to be changed when halting.


After loading the **:halt** plugin:

{% highlight ruby %}
plugin :halt
{% endhighlight %}

You can call the halt method with an integer to set the response status and return:

{% highlight ruby %}  
route do |r|
  
  r.halt(403)
  
end
{% endhighlight %}


Or set the response body and return:


{% highlight ruby %}
route do |r|
  
  r.halt('body')
  
end
{% endhighlight %}


Or set both:


{% highlight ruby %}
route do |r|
  
  r.halt(403, 'body')
  
end
{% endhighlight %}

Or set response status, headers, and body:

{% highlight ruby %}
route do |r|
  
  r.halt(403, { 'Content-Type': 'text/csv' }, 'body')
  
end
{% endhighlight %}


Note that there is a difference between provide `status`, `headers`, and `body` as separate arguments 
and providing them as a single Rack response array. 
 
With a `Rack` `response` array, the values are used directly, while with three (3) arguments, the 
`headers` given are merged into the existing `headers` and the given `body` is written to the existing 
`response body`.

If using other plugins that recognise additional types of match block responses, such as 
**:symbol_views** and **:json**, you can pass those additional types to `#.r.halt`:


{% highlight ruby %}
  plugin :halt
  plugin :symbol_views
  plugin :json
  
  route do |r|
    
    r.halt(:template)
    
    r.halt(500, [{ 'error': 'foo'}])
    
    r.halt(500, header: 'value', :other_template)
  
  end
{% endhighlight %}

Note that when using the **:json** plugin with the **:halt** plugin, you cannot return a array as a single 
argument and have it be converted to JSON, since it would be interpreted as a Rack response.  

You must use call `r.halt` with either two or three argument forms in that case.


### RequestMethods


#### `#.halt(*res)`

Expand default halt method to handle status codes, headers, and bodies.  See Halt.




