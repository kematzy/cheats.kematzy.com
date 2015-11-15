---
title: [Ruby, Roda - Web Framework, Plugins, ":json_parser"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:json_parser** plugin parses request bodies in JSON format if the request's content type specifies JSON. 

This is mostly designed for use with JSON API sites.

This only parses the request body as JSON if the `Content-Type` header for the request includes 'json'.


### Configurations

Handle options for the **:json_parser** plugin:

| **Option Name:** | **Description:** |
| --- | --- |
| :error_handler      | A `proc` to call if an `exception` is raised when parsing a JSON request body. The `proc` is called with the `request` object, and should probably call `halt` on the `request` or raise an `exception`. |
| :parser             | The parser to use for parsing incoming JSON. Should be an object that responds to `call(str)` and returns the parsed data.  The default is to call `JSON.parse`. |
| :include_request    | If `true`, the parser will be called with the `request` object as the second argument, so the parser needs to respond to `call(str, request)`. |


{% highlight ruby %}
def self.configure(app, opts=OPTS)
  app.opts[:json_parser_error_handler] = opts[:error_handler] || app.opts[:json_parser_error_handler] || DEFAULT_ERROR_HANDLER
  app.opts[:json_parser_parser] = opts[:parser] || app.opts[:json_parser_parser] || DEFAULT_PARSER
  app.opts[:json_parser_include_request] = opts[:include_request] || app.opts[:json_parser_include_request]
end
{% endhighlight %}


---

### RequestMethods


#### `#.POST`

If the `Content-Type` header in the request includes `'json'`, parse the request body as JSON.  
Ignore an empty request body.


