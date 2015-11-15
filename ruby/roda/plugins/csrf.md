---
title: [Ruby, Roda - Web Framework, Plugins, ":csrf"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:csrf** plugin adds CSRF protection using the `rack_csrf` gem, along with some `csrf` helper methods 
to use in your views.  


To use it, load the plugin, with the options hash passed to `Rack::Csrf`:


{% highlight ruby %}
plugin :csrf, raise: true
{% endhighlight %}


This adds the following instance methods:

| **Method Name:** | **Description:** |
| --- | --- |
| csrf_field          | The field name to use for the hidden/meta `csrf` tag. |
| csrf_header         | The http header name to use for submitting `csrf` token via headers (useful for JavaScript). |
| csrf_metatag        | An html meta tag string containing the token, suitable for placing in the page header |
| csrf_tag            | An html hidden input tag string containing the token, suitable for placing in an html form. |
| csrf_token          | The value of the `csrf` token, in case it needs to be accessed directly. |


### InstanceMethods

#### `#.csrf_field`

The name of the hidden/meta `csrf` tag.


#### `#.csrf_header`

The http header name to use for submitting `csrf` token via headers.


#### `#.csrf_metatag(opts={})`

An html meta tag string containing the token.


#### `#.csrf_tag`

An html hidden input tag string containing the token.


#### `#.csrf_token`
The value of the `csrf` token.


