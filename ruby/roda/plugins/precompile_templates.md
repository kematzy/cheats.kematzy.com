---
title: [Ruby, Roda - Web Framework, Plugins, ":precompile_templates"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:precompile_templates** plugin adds support for precompiling template code. 

This can result in a large memory savings for applications that have large templates or a large number 
of small templates if the application uses a forking webserver.  

By default, template compilation is lazy, so all the child processes in a forking webserver will have 
their own copy of the compiled template.  

By using the **:precompile_templates** plugin, you can precompile the templates in the parent process 
before forking, and then all of the child processes can use the same precompiled templates, which
saves memory.


After loading the plugin, you can call `#.precompile_templates` with the pattern of templates you would 
like to precompile:


{% highlight ruby %}
plugin :precompile_templates

precompile_templates "views/\*\*/*.erb"
{% endhighlight %}


That will precompile all `ERB` template files in the views directory or any subdirectory.

If the templates use local variables, you need to specify which local variables to precompile, 
which should be an array of symbols:

{% highlight ruby %}
  precompile_templates 'views/users/_*.erb', locals: [:user]
{% endhighlight %}


**Note!** If you have multiple local variables and are not using a [Tilt](https://github.com/tilt/tilt/) 
version greater than 2.0.1, you should specify the `:locals` option in the same order as the keys in the 
`:locals` hash you pass to render/view.  

Since hashes are not ordered in ruby 1.8, you should not attempt to precompile templates that use 
`:locals` on ruby 1.8 unless you are using a Tilt version greater than 2.0.1.  


If you are running the Tilt master branch, you can force sorting of locals using the `:sort_locals` option 
when loading the plugin.

You can specify other `render` options when calling `precompile_templates`, including `:cache_key`, 
`:template_class`, and `:template_opts`.  

If you are passing any of those options to render/view for the template, you should pass the same 
options when precompiling the template.

To compile inline templates, just pass a single hash containing an `:inline` to `precompile_templates`:

{% highlight ruby %}
  precompile_templates inline: some_template_string
{% endhighlight %}


---

### ClassMethods

#### `#.precompile_templates(pattern, opts=OPTS)`

Precompile the templates using the given options.  See `PrecompileTemplates` for details.


