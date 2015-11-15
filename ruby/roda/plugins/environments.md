---
title: [Ruby, Roda - Web Framework, Plugins, ":environments"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:environments** plugin adds a environment class accessor to get the environment for the application, 
three (3) predicate class methods to check for the current environment:

* `#.development?`
* `#.test?`
* `#.production?`

and a class configure method that takes environment(s) and yields to the block if the given 
environment(s) match the current environment.

The default environment for the application is based on `ENV['RACK_ENV']`.

### Example:

{% highlight ruby %}
class Roda
  plugin :environments

  environment  # => :development
  development? # => true
  test?        # => false
  production?  # => false

  
  # Set the environment for the application
  self.environment = :test 
  test?        # => true

  configure do
    # called, as no environments given
  end

  configure :development, :production do
    # not called, as no environments match
  end
  
  configure :test do
    # called, as environment given matches current environment
  end
  
end
{% endhighlight %}


### Configuration

Set the environment to use for the app.  Default to `ENV['RACK_ENV']` if no environment is given.  

If `ENV['RACK_ENV']` is not set and no environment is given, assume the development environment.

{% highlight ruby %}
def self.configure(app, env=ENV["RACK_ENV"])
  app.environment = (env || 'development').to_sym
end
{% endhighlight %}


### ClassMethods


#### `#.configure(*envs)`

If no environments are given or one of the given environments matches the current environment, yield 
the receiver to the block.


#### `#.environment`

The current environment for the application, which should be stored as a symbol.


#### `#.environment=(v)`

Override the environment for the application, instead of using `RACK_ENV`.


#### `#.development?`

Returns true if `ENV['RACK_ENV'] == 'development'`, else returns false


#### `#.test?`

Returns true if `ENV['RACK_ENV'] == 'test'`, else returns false


#### `#.production?`

Returns true if `ENV['RACK_ENV'] == 'production'`, else returns false


