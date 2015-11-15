---
title: [Ruby, Roda - Web Framework, Plugins, ":caching"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The caching plugin adds methods related to HTTP caching.

For proper caching, you should use either the `#.last_modified` or `#.etag` request methods.


{% highlight ruby %}
r.get '/albums/:d' do |album_id|
  @album = Album[album_id]
  r.last_modified @album.updated_at
  view('album')
end

# or

r.get '/albums/:d' do |album_id|
  @album = Album[album_id]
  r.etag @album.sha1
  view('album')
end
{% endhighlight %}


Both `last_modified` or `etag` will immediately halt processing if there have been no modifications 
since the last time the client requested the resource, assuming the client uses the appropriate 
`HTTP 1.1` request headers.


This plugin also includes the `#.cache_control` and `#.expires` response methods.  

The `#.cache_control` method sets the `Cache-Control` header using the given hash:


{% highlight ruby %}
  response.cache_control(public: true, max_age: 60)
    # Cache-Control: public, max-age=60
{% endhighlight %}


The `#.expires` method is similar, but in addition to setting the `HTTP 1.1 Cache-Control` header, 
it also sets the `HTTP 1.0 Expires` header:


{% highlight ruby %}
 response.expires(60, public: true)
 # Cache-Control: public, max-age=60
 # Expires: Mon, 29 Sep 2014 21:25:47 GMT
{% endhighlight %}

The implementation was originally taken from [Sinatra](http://sinatrarb.com/), which is also released under the MIT License:

Copyright (c) 2007, 2008, 2009 Blake Mizerany 
<br>
Copyright (c) 2010, 2011, 2012, 2013, 2014 Konstantin Haase

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and 
associated documentation files (the "Software"), to deal in the Software without restriction, 
including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, 
and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, 
subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial 
portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT 
LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. 
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE 
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

### RequestMethods


#### `#.last_modified(time)`

Set the last modified time of the resource using the `Last-Modified` header. The `time` argument 
should be a `Time` instance.

If the current request includes an `If-Modified-Since` header that is equal or later than the time 
specified, immediately returns a response with a `304` status.

If the current request includes an `If-Unmodified-Since` header that is before than the time specified, 
immediately returns a response with a `412` status.


#### `#.etag(value, opts=OPTS)`

Set the response entity tag using the `ETag` header.

The `value` argument is an identifier that uniquely identifies the current version of the resource.


| **Options:** | |
| --- | --- |
| :weak           | Use a weak cache validator (a strong cache validator is the default) |
| :new_resource   | Whether this etag should match an etag of * (true for POST, false otherwise) |

When the current request includes an If-None-Match header with a matching `etag`, immediately returns 
a response with a `304` or `412` status, depending on the request method.

When the current request includes an If-Match header with an `etag` that doesn't match, immediately 
returns a response with a `412` status.


### ResponseMethods


#### `#.cache_control(opts)`

Specify response freshness policy for using the Cache-Control header.

Options can can any non-value directives (`:public`, `:private`, `:no_cache`, `:no_store`, 
`:must_revalidate`, `:proxy_revalidate`),  with true as the value. Options can also contain value 
directives (`:max_age`, `:s_maxage`).

{% highlight ruby %}
  response.cache_control(public: true, max_age: 60)
    # => Cache-Control: public, max-age=60
{% endhighlight %}

See [RFC 2616 / 14.9](http://tools.ietf.org/html/rfc2616#section-14.9.1) for more on standard cache control directives.


#### `#.expires(max_age, opts=OPTS)`

Set `Cache-Control` header with the `max_age` given.  `max_age` should be an integer number of seconds 
that the current request should be cached for.  

Also sets the `Expires` header, useful if you have `HTTP 1.0` clients (`Cache-Control` is an `HTTP 1.1` header).


#### `#.finish`

Remove `Content-Type` and `Content-Length` for `304` responses.



