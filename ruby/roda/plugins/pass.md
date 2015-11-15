---
title: [Ruby, Roda - Web Framework, Plugins, ":pass"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The pass plugin adds a request +pass+ method to skip the current match block as if it did not match.

{% highlight ruby %}
plugin :pass

route do |r|
  
  r.on 'foo/:bar' do |bar|
    pass if bar == 'baz'
    "/foo/#{bar} (not baz)"
  end

  r.on 'foo/baz' do
    '/foo/baz'
  end
  
end
{% endhighlight %}


--- 

### RequestMethods

#### `#.pass`

Skip the current match block as if it did not match.

