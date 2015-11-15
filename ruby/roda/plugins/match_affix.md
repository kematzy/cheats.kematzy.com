---
title: [Ruby, Roda - Web Framework, Plugins, ":match_affix"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:match_affix** plugin allows changing the default prefix and suffix used for match patterns.  

Roda's default behaviour for a match pattern like `'albums'` is to use the pattern `/\A\/(?:albums)(?=\/|\z)/`.  

This prefixes the pattern with `/` and suffixes it with `(?=\/|\z)`.  With the **:match_affix** plugin, 
you can change the prefix and suffix to use.  

So if you want to be explicit and require a leading `/` in patterns, you can set the prefix to `""`.  

If you want to consume a trailing slash instead of leaving it, you can set the suffix to `(\/|\z)`.

You set the prefix and suffix to use by passing arguments when loading the plugin:

{% highlight ruby %}
plugin :match_affix, ""
{% endhighlight %}

will load the plugin and use an empty prefix.

{% highlight ruby %}
plugin :match_affix, "", /(\/|\z)/
{% endhighlight %}

will use an empty prefix and change the suffix to consume a trailing slash.

{% highlight ruby %}
 plugin :match_affix, nil, /(\/|\z)/
{% endhighlight %}

will not modify the prefix and will change the suffix to consume a trailing slash.

