---
title: [Ruby, Roda - Web Framework, Plugins, ":header_matchers"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:header_matchers** plugin adds hash matchers for matching on less-common `HTTP` headers.

{% highlight ruby %}
plugin :header_matchers
{% endhighlight %}


It adds a `:header` matcher for matching on arbitrary headers, which matches if the `header` is present:

{% highlight ruby %}
r.on(header: 'X-App-Token') do |header_value|
  # perform route action
end
{% endhighlight %}


It adds a `:host` matcher for matching by the host of the request:


{% highlight ruby %}
r.on(host: 'foo.example.com') do
  
end
  
r.on(host: /\A\w+.example.com/) do
  
end
{% endhighlight %}

It adds a `:user_agent` matcher for matching on a user agent patterns, which yields the regexp captures to the block:

{% highlight ruby %}
r.on(user_agent: /Chrome\/([.\d]+)/) do |chrome_version|
  
end
{% endhighlight %}



It adds an `:accept` matcher for matching based on the `Accept` header:

{% highlight ruby %}
r.on(accept: 'text/csv') do
  
end
{% endhighlight %}


Note! that the accept matcher is very simple and cannot handle wildcards, priorities, or anything but 
a simple comma separated list of mime types.

