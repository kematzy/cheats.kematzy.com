---
title: [Ruby, Gems, Incoming!]
layout: default
asset_path: ../..

---

# {{ page.title | join: " / " }}

---- 

<div id="toc"></div>

---


## Introduction

Receive email in your Rack apps.

Incoming! receives a `Rack::Request` and hands you a [`Mail::Message`](https://github.com/mikel/mail/), much
like `ActionMailer::Base.receive` does with a raw email. 


We currently support the following services:

* SendGrid
* Mailgun
* Postmark
* CloudMailin
* Mandrill
* Any mail server capable of routing messages to a system command


Brought to you by :zap: **Honeybadger.io**, painless [Rails exception tracking](https://www.honeybadger.io/).


---

## Installation

1. Add Incoming! to your Gemfile and run `bundle install`:

    {% highlight ruby %}
    gem 'incoming'
    {% endhighlight %}

2. Create a new class to receive emails (see examples below)

3. Implement an HTTP endpoint to receive HTTP post hooks, and pass the
   request to your receiver. (see examples below)


---


## SendGrid example

{% highlight ruby %}
class EmailReceiver < Incoming::Strategies::SendGrid
  def receive(mail)
    %(Got message from #{mail.to.first} with subject "#{mail.subject}")
  end
end

req = Rack::Request.new(env)
result = EmailReceiver.receive(req) # => Got message from whoever@wherever.com with subject "hello world"
{% endhighlight %}


[Sendgrid API reference](http://sendgrid.com/docs/API_Reference/Webhooks/parse.html)


---


## Mailgun example

{% highlight ruby %}
class EmailReceiver < Incoming::Strategies::Mailgun
  setup :api_key => 'asdf'

  def receive(mail)
    %(Got message from #{mail.to.first} with subject "#{mail.subject}")
  end
end

req = Rack::Request.new(env)
result = EmailReceiver.receive(req) # => Got message from whoever@wherever.com with subject "hello world"
{% endhighlight %}

[Mailgun API reference](http://documentation.mailgun.net/user_manual.html#receiving-messages)


---

## Postmark example

{% highlight ruby %}
class EmailReceiver < Incoming::Strategies::Postmark
  def receive(mail)
    %(Got message from #{mail.to.first} with subject "#{mail.subject}")
  end
end

req = Rack::Request.new(env)
result = EmailReceiver.receive(req) # => Got message from whoever@wherever.com with subject "hello world"
{% endhighlight %}

[Postmark API reference](http://developer.postmarkapp.com/developer-inbound.html)


---

## CloudMailin example

Use the Raw Format when setting up your address target.

{% highlight ruby %}
class EmailReceiver < Incoming::Strategies::CloudMailin
  def receive(mail)
    %(Got message from #{mail.to.first} with subject "#{mail.subject}")
  end
end

req = Rack::Request.new(env)
result = EmailReceiver.receive(req) # => Got message from whoever@wherever.com with subject "hello world"
{% endhighlight %}

[CloudMailin API reference](http://docs.cloudmailin.com/http_post_formats/)


---


## Mandrill example

Mandrill is capable of sending multiple events in a single webhook, so the Mandrill strategy works a 
bit differently than the others. 

Namely, the `.receive` method returns an Array of return values from your `#receive` method for each 
inbound event in the payload. Otherwise, the implementation is the same:

{% highlight ruby %}
class EmailReceiver < Incoming::Strategies::Mandrill
  def receive(mail)
    %(Got message from #{mail.to.first} with subject "#{mail.subject}")
  end
end

req = Rack::Request.new(env)
result = EmailReceiver.receive(req) # => ['Got message from whoever@wherever.com with subject "hello world"', '...']
{% endhighlight %}

[Mandrill API reference](http://help.mandrill.com/entries/22092308-What-is-the-format-of-inbound-email-webhooks-)


---


## Postfix example

{% highlight ruby %}
class EmailReceiver < Incoming::Strategies::HTTPPost
  setup :secret => '6d7e5337a0cd69f52c3fcf9f5af438b1'

  def receive(mail)
    %(Got message from #{mail.to.first} with subject "#{mail.subject}")
  end
end

req = Rack::Request.new(env)
result = EmailReceiver.receive(req) # => Got message from whoever@wherever.com with subject "hello world"
{% endhighlight %}


{% highlight ruby %}
  # /etc/postfix/virtual
  @example.com http_post

  # /etc/mail/aliases
  http_post: "|http_post -s 6d7e5337a0cd69f52c3fcf9f5af438b1 http://www.example.com/emails"
{% endhighlight %}


---


## Qmail example:

{% highlight ruby %}
class EmailReceiver < Incoming::Strategies::HTTPPost
  setup :secret => '6d7e5337a0cd69f52c3fcf9f5af438b1'

  def receive(mail)
    %(Got message from #{mail.to.first} with subject "#{mail.subject}")
  end
end

req = Rack::Request.new(env)
result = EmailReceiver.receive(req) # => Got message from whoever@wherever.com with subject "hello world"
{% endhighlight %}


To setup a *global* incoming email alias:

{% highlight ruby %}
 # /var/qmail/alias/.qmail-whoever - mails to whoever@ will be delivered to this alias.
 |http_post -s 6d7e5337a0cd69f52c3fcf9f5af438b1 http://www.example.com/emails
{% endhighlight %}


Domain-specific incoming aliases can be set as follows:


{% highlight ruby %}
 #/var/qmail/control/virtualdomains
 example.com:example

 #~example/.qmail-whoever
 |http_post -s 6d7e5337a0cd69f52c3fcf9f5af438b1 http://www.example.com/emails
{% endhighlight %}


Now mails to `whoever@example.com` will be posted to the corresponding URL above. To post all mails for `example.com`, just add the above line to `~example/.qmail-default`.

---

## Example Rails controller

{% highlight ruby %}
 # app/controllers/emails_controller.rb
 class EmailsController < ActionController::Base
   def create
     if EmailReceiver.receive(request)
       render :json => { :status => 'ok' }
     else
       render :json => { :status => 'rejected' }, :status => 403
     end
   end
 end
{% endhighlight %}


{% highlight ruby %}
 # config/routes.rb
 Rails.application.routes.draw do
   post '/emails' => 'emails#create'
 end
{% endhighlight %}


{% highlight ruby %}
 # spec/controllers/emails_controller_spec.rb
 require 'spec_helper'
 
 describe EmailsController, '#create' do
   it 'responds with success when request is valid' do
     EmailReceiver.should_receive(:receive).and_return(true)
     post :create
     response.should be_success
     response.body.should eq '{"status":"ok"}'
   end
 
   it 'responds with 403 when request is invalid' do
     EmailReceiver.should_receive(:receive).and_return(false)
     post :create
     response.status.should eq 403
     response.body.should eq '{"status":"rejected"}'
   end
 end
{% endhighlight %}


---


## TODO

1. Provide authentication for all strategies where possible (currently
   only Mailgun requests are authenticated.)

--- 

## License

Incoming! is Copyright 2013 © Joshua Wood and Honeybadger Industries LLC. It is free software, and
may be redistributed under the terms specified in the MIT-LICENSE file.

