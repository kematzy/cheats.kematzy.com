---
title: [Ruby, Roda - Web Framework, Plugins, ":run_handler"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:run_handler** plugin allows `r.run` to take a block, which is yielded the Rack response array, 
before it returns it as a response.

Additionally, `r.run` also takes a options hash, and you can provide a `not_found: :pass` option to 
keep routing normally if the Rack app returns a `404` response.

{% highlight ruby %}
  plugin :run_handler

  route do |r|
    
    r.on 'a' do
      # Keep running code if RackAppFoo doesn't return a result
      r.run RackAppFoo, not_found: :pass

      # Change response status codes before returning.
      r.run(RackAppBar) do |response|
        response[0] = 200 if response[0] == 201
      end
      
    end
    
  end
{% endhighlight %}



--- 

### RequestMethods

### `#.run(app, opts=OPTS)`

If a block is given, yield the Rack `response` array to it.  The `response` can be modified before it is 
returned by the current app.

If the `{ not_found: :pass }` option is given, and the Rack `response` returned by the app is a `404` 
`response`, do not return the response, continue routing normally.


