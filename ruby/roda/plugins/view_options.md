---
title: [Ruby, Roda - Web Framework, Plugins, ":view_options"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:view_options** plugin allows you to override view and layout options and locals for specific 
branches and routes.

{% highlight ruby %}
  plugin :render
  plugin :view_options

  route do |r|
    r.on "users" do
      set_layout_options :template=>'users_layout'
      set_layout_locals :title=>'Users'
      set_view_options :engine=>'haml'
      set_view_locals :footer=>'(c) Roda'

      # ...
    end
  end
{% endhighlight %}


The `options` and `locals` you specify have higher precedence than the **:render** plugin options, 
but lower precedence than options you directly pass to the view/render methods.


## View Subdirectories

The **:view_options** plugin also has special support for sites that have outgrown a flat view 
directory and use subdirectories for views.  

It allows you to set the view directory to use, and **template names that do NOT contain a slash will
automatically use that view subdirectory**.  


### Example:

{% highlight ruby %}

  plugin :render, :layout=>'./layout'
  plugin :view_options

  route do |r|
    r.on "users" do
      set_view_subdir 'users'
      
      r.get :id do
        append_view_subdir 'profile'
        view 'index' # uses ./views/users/profile/index.erb
      end

      r.get 'list' do
        view 'lists/users' # uses ./views/lists/users.erb
      end
    end
  end

{% endhighlight %}


**Note!**  When a view subdirectory is set, **the layout will also be looked up in the subdirectory 
<u>unless it contains a slash</u>**.

So if you want to use a view subdirectory for templates but have a shared layout, you should make sure your
layout contains a slash, similar to the example above.



## Per-branch HTML escaping

If you have an existing `Roda` application that doesn't use automatic HTML escaping for `<%= %>` tags 
via the **:render** plugin's `:escape` option, but you want to switch to using the `:escape` option, 
you can now do so without making all changes at once.  

With `#.set_view_options`, you can now specify escaping or not on a per branch basis in the routing tree:

{% highlight ruby %}
  plugin :render, escape: true
  plugin :view_options
  
  route do |r|
  
    # Don't escape <%= %> by default
    set_view_options :template_opts=>{:engine_class=>nil}

    r.on "users" do
      
      # Escape <%= %> in this branch
      set_view_options :template_opts=>{:engine_class=>render_opts[:template_opts][:engine_class]}
      
    end
    
  end

{% endhighlight %}





--- 

### InstanceMethods


The following methods are created via metaprogramming:

| **Method Name:** | **Description:** |
| --- | --- |
| set_layout_locals   | Set locals to use in the layout |
| set_layout_options  | Set options to use when rendering the layout |
| set_view_locals     | Set locals to use in the view |
| set_view_options    | Set options to use when rendering the view |


#### `#.append_view_subdir(v)`

Append a view subdirectory to use.  If there hasn't already been a view subdirectory set, this just 
sets it to the argument. If there has already been a view subdirectory set, this sets the view 
subdirectory to a subdirectory of the existing view subdirectory.


#### `#.set_view_subdir(v)`

Set the view subdirectory to use.  This can be set to `nil` to not use a view subdirectory.


