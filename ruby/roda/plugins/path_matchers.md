---
title: [Ruby, Roda - Web Framework, Plugins, ":path_matchers"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:path_matchers** plugin adds hash matchers that operate on the request's path.

It adds a `:prefix` matcher for matching on the path's prefix, yielding the rest of the matched segment:

{% highlight ruby %}
r.on(prefix: 'foo') do |suffix|
  
  # Matches '/foo-bar', yielding '-bar'
  # Does not match bar-foo
  
end
{% endhighlight %}



It adds a `:suffix` matcher for matching on the path's suffix, yielding the part of the segment before the suffix:


{% highlight ruby %}
r.on(suffix: 'bar') do |prefix|

  # Matches '/foo-bar', yielding 'foo-'
  # Does not match bar-foo
  
end
{% endhighlight %}


It adds an `:extension` matcher for matching on the given file extension, yielding the part of the 
segment before the extension:


{% highlight ruby %}
r.on(extension: 'bar') do |reset|
  
  # Matches '/foo.bar', yielding 'foo'
  # Does not match bar.foo
  
end
{% endhighlight %}



---

### RequestMethods

#### `#.match_extension(ext)`

Match when the current segment ends with the given extension. request path end with the extension.


#### `#.match_prefix(prefix)`

Match when the current path segment starts with the given prefix.


#### `#.match_suffix(suffix)`

Match when the current path segment ends with the given suffix.


