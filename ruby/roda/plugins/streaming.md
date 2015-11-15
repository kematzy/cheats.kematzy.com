---
title: [Ruby, Roda - Web Framework, Plugins, ":streaming"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:streaming** plugin adds support for streaming responses from roda using the `#.stream` method:

{% highlight ruby %}
plugin :streaming

route do |r|
  
  stream do |out|
    
    ['a', 'b', 'c'].each{|v| out << v; sleep 1}
    
  end
  
end
{% endhighlight %}


In order for streaming to work, **any webservers used in front of the roda app must not buffer responses**.

The stream method takes the following options:

| **Option Name:** | **Description:** |
| --- | --- |
| :callback       | A callback proc to call when the connection is closed. |
| :keep_open      | Whether to keep the connection open after the stream block returns, default is false. |
| :loop           | Whether to call the stream block continuously until the connection is closed. |


### License

The implementation was originally taken from [Sinatra](http://sinatrarb.com/), which is also released 
under the MIT License:

Copyright (c) 2007, 2008, 2009 Blake Mizerany
<br>
Copyright (c) 2010, 2011, 2012, 2013, 2014 Konstantin Haase

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and 
associated documentation files (the "Software"), to deal in the Software without restriction, including 
without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to 
the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial 
portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT 
LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. 
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE 
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.


---

### InstanceMethods

#### `#.stream(opts=OPTS, &block)`

Immediately return a streaming response using the current response status and headers, calling the 
block to get the streaming response. See Streaming for details.

