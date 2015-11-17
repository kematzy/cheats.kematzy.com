---
title: [Ruby, Gems, Rack::Test]
layout: default
asset_path: ../..

---

# {{ page.title | join: " / " }}

---- 

<div id="toc"></div>

---


## Introduction

Rack::Test [<img src="https://codeclimate.com/github/brynary/rack-test.png" />](https://codeclimate.com/github/brynary/rack-test)


Rack::Test is a small, simple testing API for Rack apps. It can be used on its own or as a reusable 
starting point for Web frameworks and testing libraries to build on. Most of its initial 
functionality is an extraction of Merb 1.0's request helpers feature.


## Features

* Maintains a cookie jar across requests
* Easily follow redirects when desired
* Set request headers to be used by all subsequent requests
* Small footprint. Approximately 200 LOC


### Examples

{% highlight ruby %}
require 'rack/test'

class HomepageTest < Test::Unit::TestCase
  include Rack::Test::Methods
  
  def app
    MyApp.new
  end
  
  def test_redirect_logged_in_users_to_dashboard
    authorize 'bryan', 'secret'
    get '/'
    follow_redirect!
    
    assert_equal "http://example.org/redirected", last_request.url
    assert last_response.ok?
  end
  
end
{% endhighlight %}


If you want to test one app in isolation, you just return that app as shown above. But if you want 
to test the entire app stack, including middlewares, cascades etc. you need to parse the app 
defined in config.ru.

{% highlight ruby %}
OUTER_APP = Rack::Builder.parse_file('config.ru').first

class TestApp < Test::Unit::TestCase
  include Rack::Test::Methods

  def app
    OUTER_APP
  end

  def test_root
    get '/'
    assert last_response.ok?
  end
  
end
{% endhighlight %}


## Install


To install the latest release as a gem:

{% highlight bash %}
  gem install rack-test
{% endhighlight %}


Or via Bundler:

{% highlight ruby %}
  gem "rack-test", require: "rack/test"
{% endhighlight %}



