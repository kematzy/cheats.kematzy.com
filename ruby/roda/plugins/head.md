---
title: [Ruby, Roda - Web Framework, Plugins, ":head"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction

The **:head** plugin attempts to automatically handle `HEAD` requests, by treating them as `GET` requests 
and returning an empty body without modifying the response status or response headers.

So for the following routes:

{% highlight ruby %}
route do |r|
  
  r.root do
    'root'
  end
  
  r.get 'a' do
    'a'
  end
  
  r.is 'b', :method => [:get, :post] do
    'b'
  end
  
end
{% endhighlight %}


`HEAD` requests for `/`, `/a`, and `/b` will all return `200` status with an empty body.


**NOTE: if you have a public facing website it is recommended that you enable this plugin.** 

Search engines and other bots may send a `HEAD` request prior to crawling a page with a `GET` request. 
Without this plugin those `HEAD` requests will return a `404` status, which may prevent search engine's 
from crawling your website.


--- 

### InstanceMethods


#### `#.call(*)`

Always use an empty response body for head requests, with a content length of 0.



---

### RequestMethods


#### `#.is_get?`

Consider `HEAD` requests as `GET` requests.


