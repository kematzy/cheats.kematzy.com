---
title: [Ruby, Roda - Web Framework, Plugins, ":chunked"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:chunked** plugin allows you to stream responses to clients using `Transfer-Encoding: chunked`.  

This can significantly improve performance of page rendering on the client, as it flushes the headers 
and top part of the layout template (generally containing references to the stylesheet and javascript assets) 
before rendering the content template.

This allows the client to fetch the assets while the template is still being rendered.  

Additionally, this plugin makes it easy to defer executing code required to render the content template 
until after the top part of the layout has been flushed, so the client can fetch the assets while the 
application is still doing the necessary processing in order to render the content template, such as 
retrieving values from a database.

There are a couple disadvantages of streaming using chunked encoding. 

First is that the layout must be rendered before the content, so any state changes made in your content 
template will not affect the layout template.


Second, error handling is reduced, since if an error occurs while rendering a template, a successful 
response code has already been sent.


To use chunked encoding for a response, just call the `#.chunked` method instead of view:

{% highlight ruby %}
r.root do
  
  chunked(:index)
  
end
{% endhighlight %}



If you want to execute code after flushing the top part of the layout template, but before rendering 
the content template, pass a block to chunked:

{% highlight ruby %}
r.root do
  
  chunked(:index) do
    # expensive calculation here
  end
  
end
{% endhighlight %}

You can also call delay manually with a block, and the execution of the block will be delayed until 
rendering the content template.  


This is useful if you want to delay execution for all routes under a branch:

{% highlight ruby %}
r.on 'albums/:d' do |album_id|
  
  delay do
    @album = Album[album_id]
  end
  
  r.get 'info' do
    chunked(:info)
  end
  
  r.get 'tracks' do
    chunked(:tracks)
  end
  
end
{% endhighlight %}

If you want to chunk all responses, pass the `:chunk_by_default` option when loading the plugin:

{% highlight ruby %}
  plugin :chunked, chunk_by_default: true
{% endhighlight %}


then you can just use the normal view method:

{% highlight ruby %}
  r.root do
    view(:index)
  end
{% endhighlight %}

and it will chunk the response.  


Note that you still need to call `#.chunked` if you want to pass a block of code to be executed after 
flushing the layout and before rendering the content template. 

Also, before you enable chunking by default, you need to make sure that none of your content templates 
make state changes that affect the layout template. 

Additionally, make sure nowhere in your app are you doing any processing after the call to view.


If you use `:chunk_by_default`, but want to turn off chunking for a view, call `#.no_chunk!`:

{% highlight ruby %}
r.root do

  no_chunk!
  view(:index)
  
end
{% endhighlight %}



Inside your layout or content templates, you can call the flush method to flush the current result 
of the template to the user, useful for streaming large datasets.

{% highlight ruby %}
<% (1..100).each do |i| %>
  <%= i %>
  <% sleep 0.1 %>
  <% flush %>
<% end %>
{% endhighlight %}


Note that you should not call flush from inside sub-templates of the content or layout templates, 
unless you are also calling flush directly before rendering the sub-template, and also directly 
injecting the sub-template into the current template without modification.  

So if you are using the above template code in a sub-template, in your content template you should do:

{% highlight ruby %}
<% flush %><%= render(:subtemplate) %>
{% endhighlight %}

If you want to use chunked encoding when rendering a template, but don't want to use a layout, 
pass the `layout: false` option to chunked.

{% highlight ruby %}
r.root do

  chunked(:index, layout: false)
  
end
{% endhighlight %}


In order to handle errors in chunked responses, you can override the `#.handle_chunk_error` method:

{% highlight ruby %}
def handle_chunk_error(e)
  env['rack.logger'].error(e)
end
{% endhighlight %}


It is possible to set `@_out_buf` to an error notification and call `#.flush` to output the message to the 
client inside `handle_chunk_error`.

In order for chunking to work, you must make sure that no proxies between the application and the 
client buffer responses.  


Also, this plugin only works for `HTTP/1.1` requests since `Transfer-Encoding: chunked` is not supported 
in `HTTP/1.0`.  If an `HTTP/1.0` request is submitted, this plugin will automatically fallback to the 
normal template rendering.

Note that some proxies including `nginx` default to `HTTP/1.0` even if the client supports `HTTP/1.1`.  
For `nginx`, set the `proxy_http_version` to `1.1`.

If you are using `nginx` and have it set to buffer proxy responses by default, you can turn this off 
on a per response basis using the `X-Accel-Buffering` header.  

To set this header or similar headers for all chunked responses, pass a :headers option when loading the plugin:

{% highlight ruby %}
  plugin :chunked, headers: { 'X-Accel-Buffering' => 'no' }
{% endhighlight %}


The **:chunked** plugin requires the render plugin, and only works for template engines that store 
their template output variable in `@_out_buf`.  

Also, it only works if the content template is directly injected into the layout template without modification.

If using the chunked plugin with the **:flash** plugin, make sure you call the `#.flash` method early 
in your route block.  If the `#.flash` method is not called until template rendering, the flash may not be
rotated.



### InstanceMethods


#### `#.no_chunk!`

Disable chunking for the current request.  Mostly useful when chunking is turned on by default.


#### `#.view(*a)`

If chunking by default, call chunked if it hasn't yet been called and chunking is not specifically disabled.


#### `#.chunked(template, opts=OPTS, &block)`

Render a response to the user in chunks.  See Chunked for an overview.  If a block is given, it is 
passed to #delay.
          


#### `#.delay(&block)`

Delay the execution of the block until right before the content template is to be rendered.

#### `#.each_chunk`

Yield each chunk of the template rendering separately.


#### `#.handle_chunk_error(e)`

By default, raise the exception.


#### `#.flush`

Call the flusher if one is defined.  If one is not defined, this is a no-op, so flush can be used 
inside views without breaking things if chunking is not used.


