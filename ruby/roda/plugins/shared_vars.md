---
title: [Ruby, Roda - Web Framework, Plugins, ":shared_vars"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:shared_vars** plugin adds a shared method for storing shared variables across nested `Roda` apps.

{% highlight ruby %}
class API < Roda

  plugin :shared_vars
  
  route do |r|
    
    user = shared[:user]
    # ...
    
  end
  
end

class App < Roda
  
  plugin :shared_vars

  route do |r|
    
    r.on :user_id do |user_id|
      
      shared[:user] = User[user_id]
      r.run API
      
    end
    
  end
  
end
{% endhighlight %}


If you pass a hash to shared, it will update the shared vars with the content of the hash:

{% highlight ruby %}
route do |r|
  
  r.on :user_id do |user_id|
    shared(user: User[user_id])
  
    r.run API
  end
  
end
{% endhighlight %}


You can also pass a block to shared, which will set the shared variables only for the given block, 
restoring the previous shared variables afterward:

{% highlight ruby %}
route do |r|
  
  r.on :user_id do |user_id|
    
    shared(user: User[user_id]) do
      r.run API
    end
    
  end
  
end
{% endhighlight %}


---

### InstanceMethods


#### `#.shared(vars=nil)`

Returns the current shared vars for the request.  These are stored in the request's environment, 
so they will be implicitly shared with other apps using this plugin.

If the `vars` argument is given, it should be a hash that will be merged into the current shared vars.

If a block is given, a `vars` argument must be provided, and it will only make the changes to the 
shared vars for the duration of the block, restoring the previous shared vars before the block returns.

