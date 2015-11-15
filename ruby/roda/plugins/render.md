---
title: [Ruby, Roda - Web Framework, Plugins, ":render"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:render** plugin adds support for template rendering using the Tilt library.  

Two methods are provided for template rendering, `#.view` (which uses the layout) and `#.render` (which does not).


{% highlight ruby %}
plugin :render

route do |r|
  
  r.is 'foo' do
    view('foo') # renders views/foo.erb inside views/layout.erb
  end

  r.is 'bar' do
    render('bar') # renders views/bar.erb
  end
  
end
{% endhighlight %}


You can provide options to the plugin method:


{% highlight ruby %}
  plugin :render, engine: 'haml', views: 'admin_views'
{% endhighlight %}


### Plugin Options

The following plugin options are supported:

| **Option Name:** | **Description:** |
| --- | --- |
| :cache                | `nil`/`false` to not cache templates (useful for development), defaults to true unless `RACK_ENV` is development to automatically use the default template cache. |
| :engine               | The Tilt engine to use for rendering, also the default file extension for templates, defaults to `'erb'`. |
| :escape               | Use Roda's Erubis escaping support, which makes `<%= %>` escape output, `<%== %>` not escape output, and handles postfix conditions inside `<%= %>` tags. |
| :escape_safe_classes  | String subclasses that should not be HTML escaped when used in `<%= %>` tags, when :escape is used. Can be an array for multiple classes. |
| :escaper              | Object used for escaping output of `<%= %>`, when :escape is used, overriding the default.  If given, object should respond to `escape_xml` with a single argument and return an output string. |
| :layout               | The base name of the layout file, defaults to `'layout'`. |
| :layout_opts          | The options to use when rendering the layout, if different from the default options. |
| :template_opts        | The Tilt options used when rendering all templates. defaults to: `{ outvar: '@_out_buf', default_encoding: Encoding.default_external }`. |
| :engine_opts          | The Tilt options to use per template engine.  Keys are engine strings, values are hashes of template options. |
| :views                | The directory holding the view files, defaults to the 'views' subdirectory of the application's `:root` option (the process's working directory by default). |


### Render/View Method Options

Most of these options can be overridden at runtime by passing options to the `view` or `render` methods:

{% highlight ruby %}
  view('foo', engine: 'html.erb')
  
  render('foo', views: 'admin_views')
{% endhighlight %}

There are additional options to `#.view` and `#.render` that are available at runtime:

| **Option Name:** | **Description:** |
| --- | --- |
| :cache          | Set to false to not cache this template, even when caching is on by default.  Set to true to force caching for this template, even when the default is to not cache (e.g. when using the :template_block option). |
| :cache_key      | Explicitly set the hash key to use when caching. |
| :content        | Only respected by +view+, provides the content to render inside the layout, instead of rendering a template to get the content. |
| :inline         | Use the value given as the template code, instead of looking for template code in a file. |
| :locals         | Hash of local variables to make available inside the template. |
| :path           | Use the value given as the full pathname for the file, instead of using the :views and :engine option in combination with the template name. |
| :template       | Provides the name of the template to use.  This allows you pass a single options hash to the render/view method, while still allowing you to specify the template name. |
| :template_block | Pass this block when creating the underlying template, ignored when using :inline.  Disables caching of the template by default. |
| :template_class | Provides the template class to use, inside of using Tilt or `Tilt[:engine]`. |


Here's how those options are used:

{% highlight ruby %}
  view(inline: '<%= @foo %>')
  
  render(path: '/path/to/template.erb')
{% endhighlight %}

If you pass a hash as the first argument to `#.view` or `#.render`, it should have either `:template`, 
`:inline`, `:path`, or `#.:content` (for `#.view`) as one of the keys.


### Speeding Up Template Rendering

By default, determining the cache key to use for the template can be a lot of work.  

If you specify the `:cache_key` option, you can save `Roda` from having to do that work, which will 
make your application faster.  However, if you do this, you need to make sure you choose a correct key.

If your application uses a unique template per path, in that the same path never uses more than one 
template, you can use the **:view_options** plugin and do:

{% highlight ruby %}
  set_view_options cache_key: r.path_info
{% endhighlight %}



at the top of your route block.  


You can even do this if you do have paths that use more than one template, as long as you specify 
`:cache_key` specifically when rendering in those paths.

If you use a single layout in your application, you can also make layout rendering faster by specifying 
`:cache_key` inside the `:layout_opts` plugin option.

--- 

### ClassMethods

#### `#.inherited(subclass)`

Copy the rendering options into the subclass, duping them as necessary to prevent changes in the 
subclass affecting the parent class.


#### `#.render_opts`

Return the render options for this class.


---

### InstanceMethods


#### `#.render(template, opts = OPTS, &block)`

Render the given template. See Render for details.


#### `#.render_opts`

Return the render options for the instance's class. While this is not currently frozen, it may be 
frozen in a future version, so you should not attempt to modify it.


#### `#.view(template, opts=OPTS)`

Render the given template.  If there is a default layout for the class, take the result of the template 
rendering and render it inside the layout.  See Render for details.


