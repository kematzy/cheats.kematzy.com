---
title: [Ruby, Roda - Web Framework, Plugins, ":json"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:json** plugin allows match blocks to return arrays or hashes, and have those arrays or hashes be
converted to JSON which is used as the response body.

It also sets the `response` content type to `application/json`. 

So you can take code like:


{% highlight ruby %}

r.root do
  response['Content-Type'] = 'application/json'
  [1, 2, 3].to_json
end

r.is 'foo' do
  response['Content-Type'] = 'application/json'
  {'a'=>'b'}.to_json
end
{% endhighlight %}


and DRY it up:


{% highlight ruby %}
plugin :json

r.root do
  [1, 2, 3]
end

r.is "foo" do
  { 'a' => 'b'}
end
{% endhighlight %}


By default, only arrays and hashes are handled, but you can specifically set the allowed classes to 
JSON by adding using the `:classes` option when loading the plugin:


{% highlight ruby %}
plugin :json, classes: [Array, Hash, Sequel::Model]
{% endhighlight %}


By default objects are serialised with `#.to_json`, but you can pass in a custom serialiser, which 
can be any object that responds to `#.call(object)`.


{% highlight ruby %}
plugin :json, serializer: proc { |o| o.to_json(root: true) }
{% endhighlight %}


If you need the request information during serialisation, such as HTTP headers or query parameters, 
you can pass in the `:include_request` option, which will pass in the request object as the second 
argument when calling the serializer.


{% highlight ruby %}
plugin :json, include_request: true, serializer: proc { |o, request| ... }
{% endhighlight %}


---

### ClassMethods

#### `#.json_result_classes`

The classes that should be automatically converted to JSON


