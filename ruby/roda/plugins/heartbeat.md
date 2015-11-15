---
title: [Ruby, Roda - Web Framework, Plugins, ":heartbeat"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:heartbeat** handles heartbeat / status requests.  

If a request for the heartbeat path comes in, a `200` response with `Content-Type: text/plain` and 
a body of `'OK'` will be returned. 

The default heartbeat path is `'/heartbeat'`, so to use that:

{% highlight ruby %}
plugin :heartbeat
{% endhighlight %}


You can also specify a custom heartbeat path:

{% highlight ruby %}
plugin :heartbeat, path: '/status'
{% endhighlight %}



---

### InstanceMethods

If the request is for a heartbeat path, return the heartbeat response.

#### `#.call`
