---
title: [Ruby, Roda - Web Framework, Plugins, ":render_each"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:render_each** plugin allows you to render a template for each value in an enumerable, returning 
the concatenation of all of the template renderings.  

For example:

{% highlight ruby %}
  render_each([1,2,3], :foo)
{% endhighlight %}

will render the `foo` template 3 times.  Each time the template is rendered, the local variable `foo` 
will contain the given value (e.g. on the first rendering `foo` is 1).

You can pass additional render options via an options hash:

{% highlight ruby %}
  render_each([1,2,3], :foo, :views=>'partials')
{% endhighlight %}

One additional option supported by is `:local`, which sets the local variable containing the current value to use.  

So:

{% highlight ruby %}
  render_each([1,2,3], :foo, local: :bar)
{% endhighlight %}

Will render the `foo` template, but the local variable used inside the template will be `bar`.  
You can use `local: nil` to not set a local variable inside the template.



---

### InstanceMethods


#### `#.render_each(enum, template, opts=OPTS)`

For each value in enum, render the given template using the given opts.  

The template and options hash are passed to `render`.

Additional options supported:

| **Option:** | **Description:** |
| --- | --- |
| :local    | The local variable to use for the current `enum` value inside the template.  An explicit `nil` value does not set a local variable.  If not set, uses the template name. |


