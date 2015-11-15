---
title: [Ruby, Roda - Web Framework, Plugins, ":mailer"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction

The **:mailer** plugin allows your `Roda` application to send emails easily.

{% highlight ruby %}
class App < Roda
  
  plugin :render
  plugin :mailer
  
  route do |r|
    
    r.on "albums" do
      
      r.mail "added" do |album|
        @album = album
        from 'from@example.com'
        to 'to@example.com'
        cc 'cc@example.com'
        bcc 'bcc@example.com'
        subject 'Album Added'
        add_file "path/to/album_added_img.jpg"
        render(:albums_added_email) # body
      end
      
    end
    
  end
  
end
{% endhighlight %}


The default method for sending a mail is `sendmail`:

{% highlight ruby %}
App.sendmail('/albums/added', Album[1])
{% endhighlight %}


If you want to return the `Mail::Message` instance for further modification, you can just use the 
`#.mail` method:

{% highlight ruby %}
mail = App.mail('/albums/added', Album[1])
mail.from 'from2@example.com'
mail.deliver
{% endhighlight %}


The **:mailer** plugin uses the mail gem, so if you want to configure how email is sent, you can use 
`Mail.defaults` (see the mail gem documentation for more details):

{% highlight ruby %}
Mail.defaults do
  delivery_method :smtp, address: 'smtp.example.com', port: 587
end
{% endhighlight %}


You can support multipart emails using `#.text_part` and `#.html_part`:

{% highlight ruby %}
r.mail "added" do |album_added|
  
  from 'from@example.com'
  to 'to@example.com'
  subject 'Album Added'
  text_part render('album_added.txt')  # views/album_added.txt.erb
  html_part render('album_added.html') # views/album_added.html.erb
  
end
{% endhighlight %}


In addition to allowing you to use Roda's render plugin for rendering email bodies, you can use all 
of Roda's usual routing tree features to DRY up your code:


{% highlight ruby %}
  r.on "albums/:d" do |album_id|
    @album = Album[album_id.to_i]
    from 'from@example.com'
    to 'to@example.com'

    r.mail "added" do
      subject 'Album Added'
      render(:albums_added_email)
    end

    r.mail "deleted" do
      subject 'Album Deleted'
      render(:albums_deleted_email)
    end
  end
{% endhighlight %}


When sending a mail via `#.mail` or `#.sendmail`, a `RodaError` will be raised if the mail object does 
not have a body. 

This is similar to the `404` status that `Roda` uses by default for web requests that don't have a body. 

If you want to specifically send an email with an empty body, you can use the explicit empty string:


{% highlight ruby %}
r.mail do
  from    'from@example.com'
  to      'to@example.com'
  subject 'No Body Here'
  ""
end
{% endhighlight %}


By default, the mailer uses `text/plain` as the `Content-Type` for emails. 

You can override the default by specifying a `:content_type` option when loading the plugin:


{% highlight ruby %}
  plugin :mailer, content_type: 'text/html'
{% endhighlight %}


The **:mailer** plugin does support being used inside a `Roda` application that is handling web requests, 
where the routing block for mails and web requests is shared.  

However, it's recommended that you create a separate Roda application for emails. This can be a subclass 
of your main `Roda` application if you want your helper methods to automatically be available in your email views.


Error raised when the using the mail class method, but the routing tree doesn't return the mail object. 

{% highlight ruby %}
class Error < ::Roda::RodaError; end
{% endhighlight %}


---

### ClassMethods


#### `#.mail(path, *args)`

Return a `Mail::Message` instance for the email for the given request path and arguments.  

You can further manipulate the returned mail object before calling `#.deliver` to send the mail.


#### `#.sendmail(*args)`

Calls `#.mail` and immediately sends the resulting mail.


---

### RequestMethods


#### `#.mail(*args)`

Similar to routing tree methods such as `#.get` and `#.post`, this matches only if the request method 
is `MAIL` (only set when using the Roda class `#.mail` or `#.sendmail` methods) and the rest of the 
arguments match the request.  

This yields any of the captures to the block, as well as any arguments passed to the `#.mail` or 
`#.sendmail` `Roda` class methods.


---

### ResponseMethods


#### `#.mail` 

The mail object related to the current request. `attr_accessor :mail`


#### `#.finish`

If the related request was an email request, add any response headers to the email, as well as adding 
the response body to the email.

Return the email unless no body was set for it, which would indicate that the routing tree did not 
handle the request.


#### `#.mail_attachments`

The attachments related to the current mail.


---

### InstanceMethods

Add delegates for common email methods.

[:from, :to, :cc, :bcc, :subject]
        
[:text_part, :html_part]


#### `#.add_file(*a, &block)`

Delay adding a file to the message until after the message body has been set.

If a block is given, the block is called after the file has been added, and you can access the 
attachment via `response.mail.attachments.last`.

