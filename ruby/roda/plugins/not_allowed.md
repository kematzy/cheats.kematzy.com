---
title: [Ruby, Roda - Web Framework, Plugins, ":not_allowed"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:not_allowed** plugin makes `Roda` attempt to automatically support the `405 Method Not Allowed` 
response status. 

The plugin changes the `r.get` and `r.post` verb methods to automatically return a 405 status if they 
are called with any arguments, and the arguments match but the request method does not match. 


So this code:

{% highlight ruby %}
r.get '' do
  'a'
end
{% endhighlight %}

will return a `200` response for `GET /` and a `405` response for `POST /`.

This plugin also changes the `r.is` method so that if you use a verb method inside `r.is`, it returns 
a `405` status if none of the verb methods match. 

So this code:

{% highlight ruby %}
  r.is '' do
    r.get do
      "a"
    end

    r.post do
      "b"
    end
  end
{% endhighlight %}

will return a `200` response for `GET /` and `POST /`, but a `405` response for `PUT /`.


**Note!** This plugin will probably not do what you want for code such as:

{% highlight ruby %}
r.get '' do
  'a'
end

r.post '' do
  'b'
end
{% endhighlight %}


Since for a `POST /` request, when +r.get+ method matches the path but not the request method, it 
will return an immediate `405` response.  


You must DRY up this code for it work correctly, like this:

{% highlight ruby %}
r.is '' do

  r.get do
    'a'
  end

  r.post do
    'b'
  end
  
end
{% endhighlight %}



In all cases where it uses a `405` response, it also sets the `Allow` header in the response to contain 
the request methods supported.

This plugin depends on the **:all_verbs** plugin.  It works by overriding the verb methods, so it 
wouldn't work if loaded after the **:all_verbs** plugin.


---
