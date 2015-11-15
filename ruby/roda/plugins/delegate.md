---
title: [Ruby, Roda - Web Framework, Plugins, ":delegate"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction

The **:delegate** plugin allows you to easily setup instance methods in the scope of the route block 
that call methods on the related `request`, `response`, or `class` which may offer a simpler API in some cases.


Roda doesn't automatically setup such delegate methods because it pollutes the application's method 
namespace, but this plugin allows the user to do so.

Here's an example based on the README's initial example, using the `#.request_delegate` method to simplify the DSL:

{% highlight ruby %}

plugin :delegate
request_delegate :root, :on, :is, :get, :post, :redirect

route do |r|
  
  # GET / request
  root do
    redirect "/hello"
  end

  # /hello branch
  on "hello" do
    # Set variable for all routes in /hello branch
    @greeting = 'Hello'

    # GET /hello/world request
    get "world" do
      "#{@greeting} world!"
    end

    # /hello request
    is do
      # GET /hello request
      get do
        "#{@greeting}!"
      end

      # POST /hello request
      post do
        puts "Someone said #{@greeting}!"
        redirect
      end
    end
  end
end
{% endhighlight %}



### ClassMethods

#### `#.class_delegate(*meths)`

Delegate the given methods to the class


#### `#.request_delegate(*meths)`

Delegate the given methods to the request


#### `#.response_delegate(*meths)`

Delegate the given methods to the response

