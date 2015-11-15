---
title: [Ruby, Roda - Web Framework, Plugins, ":param_matchers"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:param_matchers** plugin adds hash matchers that operate on the request's params.

It adds a `:param` matcher for matching on any param with the same name, yielding the value of the param.

{% highlight ruby %}
r.on(param: 'foo') do |value|
  # Matches '?foo=bar', '?foo='
  # Doesn't match '?bar=foo'
end
{% endhighlight %}


It adds a `:param!` matcher for matching on any non-empty param with the same name, yielding the value of the param.


{% highlight ruby %}
r.on(param!: 'foo') do |value|
  # Matches '?foo=bar'
  # Doesn't match '?foo=', '?bar=foo'
end
{% endhighlight %}


---

### RequestMethods


#### `#.match_param(key)`

Match the given parameter if present, even if the parameter is empty. Adds any match to the captures.


#### `#.match_param!(key)` 

Match the given parameter if present and not empty. Adds any match to the captures.


