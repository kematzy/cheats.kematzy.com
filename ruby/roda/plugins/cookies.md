---
title: [Ruby, Roda - Web Framework, Plugins, ":cookies"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction

The **:cookies** plugin adds response methods for handling cookies.

Currently, you can set cookies with `#.set_cookie` and delete cookies with `#.delete_cookie`:

{% highlight ruby %}
response.set_cookie('foo', 'bar')

response.delete_cookie('foo')
{% endhighlight %}



### ResponseMethods


#### `#.delete_cookie(key, value = {})`

Modify the headers to include a `Set-Cookie` value that deletes the cookie.  

A value hash can be provided to override the default one used to delete the cookie.

Example:

{% highlight ruby %}
response.delete_cookie('foo')

response.delete_cookie('foo', domain: 'example.org')
{% endhighlight %}
        

#### `#.set_cookie(key, value)`

Set the cookie with the given key in the headers.

{% highlight ruby %}
response.set_cookie('foo', 'bar')

response.set_cookie('foo', value: 'bar', domain: 'example.org')
{% endhighlight %}
