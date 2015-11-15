---
title: [Ruby, Roda - Web Framework, Plugins, ":multi_route"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:multi_route** plugin allows for multiple named routes, which the main route block can dispatch 
to by name at any point by calling `#.route`.  

If the named route doesn't handle the request, execution will continue, and if the named route does 
handle the request, the response returned by the named route will be returned.
 
In addition, this also adds the `r.multi_route` method, which will assume check if the first segment 
in the path matches a named route, and dispatch to that named route.
 
**Example:**


{% highlight ruby %} 
plugin :multi_route

route('foo') do |r|
 
 r.is 'bar' do
   '/foo/bar'
 end
 
end

route('bar') do |r|
 
 r.is 'foo' do
   '/bar/foo'
 end
 
end

route do |r|
 
 r.multi_route

 # or

 r.on "foo" do
   
   r.route 'foo'
   
 end

 r.on "bar" do
   
   r.route 'bar'
   
 end
 
end
{% endhighlight %}


**Note!** in multi-threaded code, **you should NOT attempt to add a named route after accepting requests**.
 

If you want to use the `r.multi_route` method, use string names for the named routes.  Also, you can 
provide a block to `r.multi_route` that is called if the route matches but the named route did not 
handle the request:


{% highlight ruby %} 
 r.multi_route do
   
   "default body"
   
 end
{% endhighlight %}
 

If a block is not provided to multi_route, the return value of the named route block will be used.
 
### Routing Files
 
The convention when using the **:multi_route** plugin is to have a single named route per file, and 
these routing files should be stored in a routes subdirectory in your application.  

So for the above example, you would use the following files:

``` 
routes/bar.rb
routes/foo.rb
```


### Namespace Support
 
The **:multi_route** plugin also has support for **namespaces**, allowing you to use `r.multi_route` at 
multiple levels in your routing tree.  

**Example:**
 
{% highlight ruby %} 
route('foo') do |r|
 
 r.multi_route('foo')
 
end

route('bar') do |r|
 
 r.multi_route('bar')
 
end

route('baz', 'foo') do |r|
 
 # handles /foo/baz prefix
 
end

route('quux', 'foo') do |r|
 
 # handles /foo/quux prefix
 
end

route('baz', 'bar') do |r|
 
 # handles /bar/baz prefix
 
end

route('quux', 'bar') do |r|
 
 # handles /bar/quux prefix
 
end

route do |r|
 
 r.multi_route

 # or

 r.on 'foo' do
  
   r.on('baz') { r.route('baz', 'foo') }
   
   r.on('quux') { r.route('quux', 'foo') }
   
 end

 r.on 'bar' do
   
   r.on('baz') { r.route('baz', 'bar') }
   
   r.on('quux') { r.route('quux', 'bar') }
   
 end
 
end
{% endhighlight %}



### Routing Files
 
The convention when using namespaces with the **:multi_route** plugin is to store the routing files 
in subdirectories per namespace. 

So for the above example, you would have the following routing files:

``` 
routes/bar.rb
routes/bar/baz.rb
routes/bar/quux.rb
routes/foo.rb
routes/foo/baz.rb
routes/foo/quux.rb
```


---

### ClassMethods


#### `#.freeze`

Freeze the namespaced routes so that there can be no thread safety issues at runtime.


#### `#.inherited(subclass)`

Copy the named routes into the subclass when inheriting.


#### `#.named_routes(namespace=nil)`

The names for the currently stored named routes


#### `#.named_route(name, namespace=nil) `

Return the named route with the given name.


#### `#.route(name=nil, namespace=nil, &block)`

If the given route has a name, treat it as a named route and store the route block. 

Otherwise, this is the main route, so call super.



---

### RequestClassMethods


#### `#. clear_named_route_regexp!(namespace=nil)`

Clear cached regexp for named routes, it will be regenerated the next time it is needed.

This shouldn't be an issue in production applications, but during development it's useful to support 
new named routes being added while the application is running.


#### `#. named_route_regexp(namespace=nil)`

A regexp matching any of the current named routes.


---

### RequestMethods

#### `#.multi_route(namespace=nil)`

Check if the first segment in the path matches any of the current named routes.  

If so, call that named route.  If not, do nothing.

If the named route does not handle the request, and a block is given, yield to the block.


#### `#.route(name, namespace=nil)`

Dispatch to the named route with the given name.


