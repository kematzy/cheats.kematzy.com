---
title: [Ruby, Roda - Web Framework, Plugins, ":backtracking_array"]
layout: default
asset\_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:backtracking_array** plugin changes the handling of array matchers such that if one of the array 
entries matches, but a later match argument fails, it will backtrack and try the next entry in the array.  


For example, the following match block does not match `/a/b` by default:

{% highlight ruby %}
r.is ['a', 'a/b'] do |path|
  # ...
end
{% endhighlight %}

This is because the <tt>'a'</tt> entry in the array matches, which makes the array match.  

However, the next matcher is the terminal matcher (since `r.is` was used), and since the path is not 
terminal as it still contains `/b` after matching <tt>'a'</tt>.

With the backtracking_array plugin, when the terminal matcher fails, matching will go on to the next 
entry in the array, <tt>'a/b'</tt>, which will also match.  

Since <tt>'a/b'</tt> matches the path fully, the terminal matcher also matches, and the match block yields.

