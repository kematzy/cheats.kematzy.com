---
title: [Ruby, Roda - Web Framework, Plugins, ":multi_run"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:multi_run** plugin provides the ability to easily dispatch to other Rack applications based on 
the request path prefix.

First, load the plugin:

{% highlight ruby %}
class App < Roda
  
  plugin :multi_run
  
end
{% endhighlight %}


Then, other Rack applications can register with the **:multi_run** plugin:

{% highlight ruby %}
App.run "ra", PlainRackApp

App.run "ro", OtherRodaApp

App.run "si", SinatraApp
{% endhighlight %}


Inside your route block, you can call `r.multi_run` to dispatch to all three Rack applications based 
on the prefix:


{% highlight ruby %}
App.route do |r|
  
  r.multi_run
  
end
{% endhighlight %}


This will dispatch routes starting with `/ra` to `PlainRackApp`, routes starting with `/ro` to `OtherRodaApp`, 
and routes starting with `/si` to `SinatraApp`.


The **:multi\_run** plugin is similar to the **:multi\_route** plugin, with the difference being the 
**:multi\_route** plugin keeps all routing subtrees in the same `Roda` app/class, while **:multi\_run** 
dispatches to other Rack apps.  


If you want to isolate your routing subtrees, **:multi_run** is a better approach, but it does not 
let you set instance variables in the main `Roda` app and have those instance variables usable in
the routing subtrees.


---

### ClassMethods

#### `#.freeze`

Freeze the multi_run apps so that there can be no thread safety issues at runtime.


#### `#.multi_run_apps`

Hash storing rack applications to dispatch to, keyed by the prefix for the application.


#### `#.run(prefix, app)`

Add a rack application to dispatch to for the given prefix when `r.multi_run` is called.


---

### RequestClassMethods


#### `#.refresh_multi_run_regexp!`

Refresh the `#.multi_run_regexp`, using the stored route prefixes, preferring longer routes before shorter routes.


#### `#.multi_run_regexp`

Refresh the `#.multi_run_regexp` if it hasn't been loaded yet.


---

### RequestMethods

#### `#.multi_run`

If one of the stored route prefixes match the current request, dispatch the request to the stored Rack application.

