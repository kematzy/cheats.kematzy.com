---
title: [Ruby, Roda - Web Framework, Plugins, ":multi_view"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:multi\_view** plugin makes it easy to render multiple views, where the view template is the 
same as the matched element.  

It adds an `r.multi_view` method, which takes an argument that is passed to `r.get`, and should
capture a single argument, which is treated as the template name to pass to `#.view`.  

This makes it possible to pass in an array of strings, or a regexp with a single capture.

The advantage of using a regexp over an array of strings is that the regexp is generally faster and 
uses less memory.  

However, constructing the regexps is more cumbersome.  To make it easier, the **:multi\_view** plugin 
also offers a `#.multi_view_compile` class method that will take an array of view template
names and construct a regexp that can be passed to `r.multi_view`.

### Example:

{% highlight ruby %}
plugin :multi_view

route do |r|
  r.multi_view(['foo', 'bar', 'baz'])
end

 # or:

route do |r|
  r.multi_view(/(foo|bar|baz)/)
end

 # or:

regexp = multi_view_compile(['foo', 'bar', 'baz'])
route do |r|
  r.multi_view(regexp)
end

 # all are equivalent to:

route do |r|

  r.get 'foo' do
    view('foo')
  end

  r.get 'bar' do
    view('bar')
  end

  r.get 'baz' do
    view('baz')
  end
  
end
{% endhighlight %}

---

### ClassMethods

#### `#.multi_view_compile(array)`

Given the array of view template names, return a regexp that will match on any of the names and 
capture the matched name.



---

### RequestMethods


Pass the argument to `#.get`, and assume if the matchers match that there is one capture, and call 
`view` with that capture.

def multi_view(arg)
