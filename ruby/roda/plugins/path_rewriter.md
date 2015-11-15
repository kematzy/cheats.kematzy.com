---
title: [Ruby, Roda - Web Framework, Plugins, ":path_rewriter"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:path_rewriter** plugin allows you to rewrite the remaining path or the path info for requests.  

This is useful if you want to transparently treat some paths the same as other paths.

By default, `#.rewrite_path` will rewrite just the remaining path.  So only routing in the current `Roda` 
app will be affected.  

This is useful if you have other code in your app that uses `PATH_INFO` and needs to see the original 
`PATH_INFO` (for example, when using relative links).


{% highlight ruby %}
 rewrite_path '/a', '/b'
 
 # PATH_INFO '/a' => remaining_path '/b'
 # PATH_INFO '/a/c' => remaining_path '/b/c'
 
{% endhighlight %}


In some cases, you may want to override `PATH_INFO` for the rewritten paths, such as when you are 
passing the request to another Rack app.

For those cases, you can use the `path_info: true` option to `rewrite_path`.

{% highlight ruby %}
  rewrite_path '/a', '/b', :path_info => true
  # PATH_INFO '/a' => PATH_INFO '/b'
  # PATH_INFO '/a/c' => PATH_INFO '/b/c'
{% endhighlight %}


If you pass a string to `#.rewrite_path`, it will rewrite all paths starting with that string.  

You can provide a regexp if you want more complete control, such as only matching exact paths.

{% highlight ruby %}
  rewrite_path /\A\/a\z/, '/b'
  
  # PATH_INFO '/a' => remaining_path '/b'
  # PATH_INFO '/a/c' => remaining_path '/a/c', no change
{% endhighlight %}


All path rewrites are applied in order, so if a path is rewritten by one rewrite, it can be rewritten 
again by a later rewrite.  Note that `PATH_INFO` rewrites are processed before `remaining_path` rewrites.


---

### ClassMethods


Freeze the path rewrite metadata.
#### `#.freeze`

#### `#.rewrite_path(was, is, opts=OPTS)`

Record a path rewrite from path `was` to path `is`.  

| Options: | | 
| --- | --- |
| :path_info | Modify `PATH_INFO`, not just remaining path. |



---

### RequestMethods

Rewrite `remaining_path` and/or `PATH_INFO` based on the path rewrites.

#### `#.initialize(scope, env)`
