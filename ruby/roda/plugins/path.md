---
title: [Ruby, Roda - Web Framework, Plugins, ":path"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:path** plugin adds support for named paths.  

Using the `#.path` class method, you can easily create `*_path` instance methods for each named path.  
Those instance methods can then be called if you need to get the path for a `form` action, `link`, 
`redirect`, or anything else.


Additionally, you can call the `#.path` class method with a class and a block, and it will register
the class.  

You can then call the `#.path` instance method with an instance of that class, and it will `instance_exec` 
the block with the arguments provided to path.

**Example:**


{% highlight ruby %}
plugin :path

path :foo, '/foo'

path :bar do |bar|
  "/bar/#{bar.id}"
end

path Baz do |baz, *paths|
  "/baz/#{baz.id}/#{paths.join('/')}"
end

path Quux |quux, path|
  "/quux/#{quux.id}/#{path}"
end


route do |r|
  
  r.post 'foo' do
    
    r.redirect foo_path # /foo
    
  end
  
  r.post 'bar' do
    
    bar = Bar.create(r.params['bar'])
    r.redirect bar_path(bar) # /bar/1
    
  end
  
  r.post 'baz' do
    
    bar = Baz[1]
    r.redirect path(baz, 'c', 'd') # /baz/1/c/d
    
  end
  
  r.post 'quux' do
    
    bar = Quux[1]
    r.redirect path(quux, '/bar') # /quux/1/bar
    
  end
  
end
{% endhighlight %}


The `path` method accepts the following options when not called with a class:


| **Option Name:** | **Description:** |
| --- | --- |
| :add_script_name      | Prefix the path generated with `SCRIPT_NAME`. This defaults to the app's `:add_script_name` option. |
| :name                 | Provide a different name for the method, instead of using `*_path`. |
| :url                  | Create a url method in addition to the path method, which will prefix the string generated with the appropriate scheme, host, and port.  If true, creates a `*_url` method.  If a `Symbol` or `String`, uses the value as the url method name. |
| :url_only             | Do not create a path method, just a url method.|


**Note!** If `:add_script_name`, `:url`, or `:url_only` is used, will also create a `_*_path` method.  

This is necessary in order to support path methods that accept blocks, as you can't pass a block to 
a block that is `instance_execed`.


--- 

### ClassMethods

#### `#.path_classes`

Hash of recognises classes for path instance method.  Keys are classes, values are `procs`.


#### `#.freeze`

Freeze the path classes when freezing the app.


#### `#.path(name, path=nil, opts=OPTS, &block)`

Create a new instance method for the named path.  See plugin module documentation for options.

#### `#.path_block(klass)`

Return the block related to the given class, or nil if there is no block.


---

### InstanceMethods

#### `#.path(obj, *args)`

Return a path based on the class of the object.  

The object passed must have had its class previously registered with the application.  If the app's 
`:add_script_name` option is true, this prepends the `SCRIPT_NAME` to the path.



