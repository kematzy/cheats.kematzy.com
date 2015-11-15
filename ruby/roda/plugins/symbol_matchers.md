---
title: [Ruby, Roda - Web Framework, Plugins, ":symbol_matchers"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:symbol_matchers** plugin allows you do define custom regexps to use for specific symbols.  

For example, if you have a route such as:

{% highlight ruby %}
  r.on :username do
    # ...
  end
{% endhighlight %}


By default this will match all nonempty segments.  However, if your usernames must be 6-20 characters, 
and can only contain `a-z` and `0-9`, you can do:

{% highlight ruby %}
  plugin :symbol_matchers
  symbol_matcher :username, /([a-z0-9]{6,20})/
{% endhighlight %}

Then the route will only if the path is `/foobar123`, but not if it is `/foo`, `/FooBar123`, or `/foobar_123`.

Note that this feature does not apply to just symbols, but also to embedded colons in strings, so the following:


{% highlight ruby %}
  r.on "users/:username" do
    # ...
  end
{% endhighlight %}


Would match `/users/foobar123`, but not `/users/foo`, `/users/FooBar123`, or `/users/foobar_123`.

By default, this plugin sets up the following symbol matchers:


| **Matchers:** | **RegExp:** | **Description:** |
| --- | --- | --- |
| :d        | `/(\d+)/`           | a decimal segment                 |
| :format   | `/(?:\.(\w+))?/`    | an optional format/extension      |
| :opt      | `/(?:\/([^\/]+))?`  | an optional segment               |
| :optd     | `/(?:\/(\d+))?`     | an optional decimal segment       |
| :rest     | `/(.*)/`            | all remaining characters, if any  |
| :w        | `/(\w+)/`           | a alphanumeric segment            |


**Note!** Because of how segment matching works, `:format`, `:opt`, and `:optd` are only going to 
work inside of a string, like this:

{% highlight ruby %}
  r.is "album:opt" do |id| end
  # matches /album (yielding nil) and /album/foo (yielding "foo")
  # does not match /album/ or /album/foo/bar
{% endhighlight %}



---

### ClassMethods

#### `#.symbol_matcher(s, re)`

Set the regexp to use for the given symbol, instead of the default.


