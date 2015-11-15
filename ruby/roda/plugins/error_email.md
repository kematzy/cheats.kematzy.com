---
title: [Ruby, Roda - Web Framework, Plugins, ":error_email"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction

The **:error_email** plugin adds an `#.error_email` instance method that sends an email related to 
the exception.  

This is most useful if you are also using the **:error_handler** plugin:

{% highlight ruby %}
plugin :error_email, to: 'to@example.com', from: 'from@example.com'

plugin :error_handler do |e|
  error_email(e)
  'Internal Server Error'
end
{% endhighlight %}


### Options:


| **Option Name:** | **Description:** |
| --- | --- |
| :from           | The **From** address to use in the email (required) |
| :to             | The To address to use in the email (required) |
| :headers        | A hash of additional headers to use in the email (default: `{}` empty hash) |
| :host           | The SMTP server to use to send the email (default: `localhost`) |
| :prefix         | A prefix to use in the email's subject line (default: `''` no prefix) |

The subject of the error email shows the exception class and message.

The body of the error email shows the backtrace of the `error` and the `request` environment, as well the 
`request params` and `session` variables (if any).

Note that emailing on every error as shown above is only appropriate for low traffic web applications.  

For high traffic web applications, use an error reporting service instead of this plugin.


### InstanceMethods

#### `#.error_email(e)`

Send an email for the given error.


