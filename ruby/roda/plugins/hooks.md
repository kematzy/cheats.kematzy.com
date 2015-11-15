---
title: [Ruby, Roda - Web Framework, Plugins, ":hooks"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:hooks** plugin adds before and after hooks to the request cycle.

{% highlight ruby %}
plugin :hooks

before do
  request.redirect('/login') unless logged_in?
  @time = Time.now
end

after do |res|
  logger.notice("Took #{Time.now - @time} seconds")
end
{% endhighlight %}


Note that in general, *before* hooks are not needed, since you can just run code at the top of the route block:


{% highlight ruby %}
plugin :hooks

route do |r|
  
  r.redirect('/login') unless logged_in?
  # ...
  
end
{% endhighlight %}


Note that the after hook is called with the rack response array of status, headers, and body.  

If it wants to change the response, it must mutate this argument, calling `response.status=` inside
an *after* block will not affect the returned status.

However, this code makes it easier to write *after* hooks, as well as handle cases where *before* hooks 
are added after the route block.


---

### ClassMethods

#### `#.after(&block)`

Adds an *after* hook.  If there is already an after hook defined, use a `proc` that `instance_execs` the 
existing *after* `proc` and then `instance_execs` the given *after* `proc`, so that the given 
*after* `proc` always executes after the previous one.


#### `#.before(&block)`

Add a *before* hook.  If there is already a *before* hook defined, use a proc that `instance_execs` 
the given *before* `proc` and then `instance_execs` the existing *before* `proc`, so that the given
*before* `proc` always executes *before* the previous one.


---

### InstanceMethods

Before routing, execute the before hooks, and execute the after hooks before returning.

#### `#.call`
