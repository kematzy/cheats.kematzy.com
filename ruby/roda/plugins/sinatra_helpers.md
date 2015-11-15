---
title: [Ruby, Roda - Web Framework, Plugins, ":sinatra_helpers"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:sinatra_helpers** plugin ports most of the helper methods defined in `Sinatra::Helpers` to `Roda`, 
other than those helpers that were already covered by other plugins such as caching and streaming.

Unlike Sinatra, the helper methods are added to either the request or response classes instead of 
directly to the scope of the route block.  

However, for consistency with Sinatra, delegate methods are added to the scope of the route block 
that call the methods on the `request` or `response`.  

If you do not want to pollute the namespace of the route block, you should load the plugin with the 
`delegate: false` option:

{% highlight ruby %}
  plugin :sinatra_helpers, delegate: false
{% endhighlight %}



### Class Methods Added

The only class method added by this plugin is `#.mime_type`, which is a shortcut for retrieving or 
setting MIME types in Rack's MIME database:


{% highlight ruby %}
  Roda.mime_type 'csv'                          # => 'text/csv'
  Roda.mime_type 'foobar', 'application/foobar' # set
{% endhighlight %}


### Request Methods Added 

In addition to adding the following methods, this changes `#.redirect` to use a `303` response status 
code by default for HTTP 1.1 non-GET requests, and to automatically use absolute URIs if the 
`:absolute_redirects` `Roda` class option is true, and to automatically prefix redirect paths with the
script name if the +:prefixed_redirects+ Roda class option is true.

When adding delegate methods, a logger method is added to the route block scope that calls the `logger` 
method on the request.


###  back

`#.back` is an alias to referrer, so you can do:

{% highlight ruby %}
  redirect back
{% endhighlight %}



### error

`#.error` sets the response status code to `500` (or a status code you provide), and halts the request.  

It takes an optional body:

{% highlight ruby %}
  error           # 500 response, empty boby

  error 501       # 501 reponse, empty body

  error 'b'       # 500 response, 'b' body

  error 501, 'b'  # 501 response, 'b' body
{% endhighlight %}


### not_found

`#.not_found` sets the response status code to 404 and halts the request.

It takes an optional body:

{% highlight ruby %}
  not_found      # 404 response, empty body

  not_found 'b'  # 404 response, 'b' body
{% endhighlight %}


### uri

`#.uri` by default returns absolute URIs that are prefixed by the script name:

{% highlight ruby %}
  request.script_name # => '/foo'

  uri '/bar'          # => 'http://example.org/foo/bar'
{% emdhighlight %}



You can turn of the absolute or script name prefixing if you want:

{% highlight ruby %}
  uri '/bar', false        # => '/foo/bar'

  uri '/bar', true, false  # => 'http://example.org/bar'

  uri '/bar', false, false # => '/bar'
{% endhighlight %}


This method is aliased as `#.url` and `#.to`.


### send_file

This will serve the file with the given path from the file system:

{% highlight ruby %}
  send_file 'path/to/file.txt'
{% endhighlight %}


Options:

:disposition :: Set the Content-Disposition to the given disposition.
:filename :: Set the Content-Disposition to attachment (unless :disposition is set),
             and set the filename parameter to the value.
:last_modified :: Explicitly set the Last-Modified header to the given value, and
                  return a not modified response if there has not been modified since
                  the previous request.  This option requires the caching plugin.
:status :: Override the status for the response.
:type :: Set the Content-Type to use for this response.


## Response Methods Added

### body

When called with an argument or block, `#.body` sets the body, otherwise it returns the body:

{% highlight ruby %}
  body      # => []
  
  body('b') # set body to 'b'
  
  body{'b'} # set body to 'b', but don't call until body is needed
{% endhighlight %}


### body=

`#.body` sets the body to the given value:

{% highlight ruby %}
  response.body = 'v'
{% endhighlight %}

This method is not delegated to the scope of the route block, call `#.body` with an argument to set the value.


### status

When called with an argument, `#.status` sets the status, otherwise it returns the status:

{% highlight ruby %}
  status      # => 200

  status(301) # sets status to 301
{% endhighlight %}


### headers

When called with an argument, `#.headers` merges the given headers into the current headers, otherwise 
it returns the headers:

{% highlight ruby %}
  headers['Foo'] = 'Bar'
  
  headers 'Foo' => 'Bar'
{% endhighlight %}


### mime_type

`#.mime_type` just calls the Roda class method to get the `mime_type`.



### content_type

When called with an argument, `#.content_type` sets the `Content-Type` based on the argument, 
otherwise it returns the `Content-Type`.


{% highlight ruby %}
  mime_type             # => 'text/html'
  
  mime_type 'csv'       # set Content-Type to 'text/csv'
  
  mime_type :csv        # set Content-Type to 'text/csv'
  
  mime_type '.csv'      # set Content-Type to 'text/csv'
  
  mime_type 'text/csv'  # set Content-Type to 'text/csv'
{% endhighlight %}


Options:

:charset :: Set the charset for the mime type to the given charset, if the charset is
            not already set in the mime type.
:default :: Uses the given type if the mime type is not known.  If this option is not
            used and the mime type is not known, an exception will be raised.

### attachment

When called with no filename, `#.attachment` just sets the `Content-Disposition` to `attachment`.  

When called with a filename, this sets the `Content-Disposition` to attachment with the appropriate 
filename parameter, and if the filename extension is recognised, this also sets the `Content-Type` 
to the appropriate MIME type if not already set.

{% highlight ruby %}
  attachment          # set Content-Disposition to 'attachment'

  attachment 'a.csv'  # set Content-Disposition to 'attachment;filename="a.csv"',
                      # also set Content-Type to 'text/csv'
{% endhighlight %}

### status predicates

This adds the following predicate methods for checking the status:

{% highlight ruby %}
  informational?  # 100-199

  success?        # 200-299

  redirect?       # 300-399

  client_error?   # 400-499

  not_found?      # 404

  server_error?   # 500-599
{% endhighlight %}

If the status has not yet been set for the response, these will return `nil`.


== License

The implementation was originally taken from [Sinatra](http://sinatrarb.com/), which is also released under the MIT License:

Copyright (c) 2007, 2008, 2009 Blake Mizerany
<br>
Copyright (c) 2010, 2011, 2012, 2013, 2014 Konstantin Haase

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and 
associated documentation files (the "Software"), to deal in the Software without restriction, including 
without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to 
the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial 
portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT 
LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. 
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE 
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.


--- 

### RequestMethods

#### `#.back`

Alias for referrer


#### `#.error(code=500, body = nil) `

Halt processing and return the error status provided with the given code and  optional body.
If a single argument is given and it is not an integer, consider it the body and use a 500 status code.


#### `.not_found(body = nil)`

Halt processing and return a 404 response with an optional body.


#### `#.redirect(path=(no_add_script_name = true; default_redirect_path), status=default_redirect_status)`

If the `absolute_redirects` or `:prefixed_redirects` roda class options has been set, respect those and update the path.


#### `#.send_file(path, opts = OPTS)`

Use the contents of the file at +path+ as the response body.  See plugin documentation for options.


#### `#.uri(addr = nil, absolute = true, add_script_name = true)` `url`, `to`

Generates the absolute URI for a given path in the app. Takes Rack routers and reverse proxies into account.



---

### ResponseMethods

#### `#.status(value = (return @status; nil))`

Set or retrieve the response status code.


#### `#.body(value = (return @body unless block_given?; nil), &block)`

Set or retrieve the response body. When a block is given, evaluation is deferred until the body is needed.


#### `#.body=(body)`

Set the body to the given value.


#### `#.finish`

If the body is a DelayedBody, set the appropriate length for it.


#### `#.headers(hash = (return @headers; nil))`

Set multiple response headers with Hash, or return the headers if no argument is given.


#### `#.mime_type(type)`

Look up a media type by file extension in Rack's mime registry.


#### `#.content_type(type = (return @headers[CONTENT_TYPE]; nil), opts = OPTS)`

Set the Content-Type of the response body given a media type or file extension.  See plugin documentation for options.


#### `#.attachment(filename = nil, disposition=ATTACHMENT)`

Set the `Content-Disposition` to `"attachment"` with the specified filename, instructing the user 
agents to prompt to save.


#### `#.informational?`

Whether or not the status is set to 1xx. Returns nil if status not yet set.


#### `#.success?`

Whether or not the status is set to 2xx. Returns nil if status not yet set


#### `#.redirect?`

Whether or not the status is set to 3xx. Returns nil if status not yet set.


#### `#.client_error?`

Whether or not the status is set to 4xx. Returns nil if status not yet set.


#### `#.server_error?`

Whether or not the status is set to 5xx. Returns nil if status not yet set.


#### `#.not_found?`

Whether or not the status is set to 404. Returns nil if status not yet set.



--- 

### ClassMethods

#### `#.mime_type(type=(return; nil), value = nil)`

If a type and value are given, set the value in Rack's `MIME` registry.

If only a type is given, lookup the type in Rack's `MIME` registry and return it.


