---
title: [Ruby, Roda - Web Framework, Plugins, ":named_templates"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:named_templates** plugin allows you to specify templates by name, providing the template code 
to use for a given name:

{% highlight ruby %}
plugin :named_templates

template :layout do
  "<html><body><%= yield %></body></html>"
end
template :index do
  "<p>Hello <%= @user %>!</p>"
end

route do |r|
  @user = 'You'
  render(:index)
end
  # => "<html><body><p>Hello You!</p></body></html>"

{% endhighlight %}


You can provide options for the template, for example to change the engine that the template uses:

{% highlight ruby %}
template :index, engine: :str do
  "<p>Hello #{@user}!</p>"
end
{% endhighlight %}


The block you use is reevaluated on every call, allowing you to easily include additional setup logic:

{% highlight ruby %}
template :index do
  greeting = ['hello', 'hi', 'howdy'].sample
  @user.downcase!
  "<p>#{greating} <%= @user %>!</p>"
end
{% endhighlight %}
  

This plugin also works with the **:view_subdirs** plugin, as long as you prefix the template name 
with the view subdirectory:

{% highlight ruby %}
template 'main/index' do
  "<html><body><%= yield %></body></html>"
end

route do |r|
  
  set_view_subdir('main')
  @user = 'You'
  render(:index)
  
end
{% endhighlight %}



---

### ClassMethods

#### `#.freeze`

Freeze the named templates so that there can be no thread safety issues at runtime.


#### `#.template(name, options=nil, &block)`

Store a new template block and options for the given template name.


