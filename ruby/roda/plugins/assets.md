---
title: [Ruby, Roda - Web Framework, Plugins, ":assets"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction

The **:assets** plugin adds support for rendering your CSS and javascript asset files on the fly in 
development, and compiling them to a single, compressed file in production.

This uses the **:render** plugin for rendering the assets, and the **:render** plugin uses tilt internally, 
so you can use any template engine supported by tilt for you assets.  


Tilt ships with support for the following asset template engines, assuming the necessary libraries are installed:

| **Type:** | **Supported Template Engines:** |
| --- | --- |
| css | Less, Sass, Scss |
| js  | CoffeeScript |


## Usage

When loading the plugin, use the `:css` and `:js` options to set the source file(s) to use for CSS 
and Javascript assets:


{% highlight ruby %}
plugin :assets, css: 'some_file.scss', js: 'some_file.coffee'
{% endhighlight %}


This will look for the following files:

```
assets/css/some_file.scss
assets/js/some_file.coffee
```


If you want to change the paths where asset files are stored, see the Options section below.

### Serving

In your routes, call the `r.assets` method to add a route to your assets, which will make your app serve 
the rendered assets:

{% highlight ruby %}
route do |r|
  r.assets
end
{% endhighlight %}


You should generally call `r.assets` inside the route block itself, and not under any branches of the routing tree.

### Views

In your layout view, use the assets method to add links to your CSS and Javascript assets:

{% highlight ruby %}
<%= assets(:css) %>
<%= assets(:js) %>
{% endhighlight %}


You can add attributes to the tags by using an options hash:

{% highlight ruby %}
<%= assets(:css, :media => 'print') %>
{% endhighlight %}


The `#.assets` method will respect the application's `:add_script_name` option, if it set it will automatically 
prefix the path with the `SCRIPT_NAME` for the request.


## Asset Groups

The asset plugin supports groups for the cases where you have different css/js files for your front end 
and back end.  


To use asset groups, you pass a hash for the `:css` and/or `:js` options:

{% highlight ruby %}
  plugin :assets, css: { frontend: 'some_frontend_file.scss',
                         backend:  'some_backend_file.scss'  }
{% endhighlight %}

This expects the following directory structure for your assets:

```
assets/css/frontend/some_frontend_file.scss
assets/css/backend/some_backend_file.scss
```

If you want do not want to force that directory structure when using asset groups, you can use the 
`group_subdirs: false` option.

In your view code use an array argument in your call to assets:

{% highlight ruby %}
<%= assets([:css, :frontend]) %>
{% endhighlight %}



### Nesting

Asset groups also supporting nesting, though that should only be needed in fairly large applications.  

You can use a nested hash when loading the plugin:

{% highlight ruby %}
plugin :assets, css: { frontend: { dashboard: 'some_frontend_file.scss' } }
{% endhighlight %}


and an extra entry per nesting level when creating the tags.


{% highlight ruby %}
<%= assets([:css, :frontend, :dashboard]) %>
{% endhighlight %}



## Caching

The **:assets** plugin uses the caching plugin internally, and will set the `Last-Modified` header to the 
modified timestamp of the asset source file when rendering the asset.

If you have assets that include other asset files, such as using `@import` in a `.sass` file, you need to 
specify the dependencies for your assets so that the assets plugin will correctly pick up changes.  


You can do this using the `:dependencies` option to the plugin, which takes a hash where
the keys are paths to asset files, and values are arrays of paths to dependencies of those asset files:

{% highlight ruby %}
app.plugin :assets,
           dependencies: { 'assets/css/bootstrap.scss' => Dir['assets/css/bootstrap/' '**/*.scss'] }
{% endhighlight %}


## Asset Compilation

In production, you are generally going to want to compile your assets into a single file, which you 
can do by calling `#.compile_assets` after loading the plugin:


{% highlight ruby %}
plugin :assets, css: 'some_file.scss', js: 'some_file.coffee'

compile_assets
{% endhighlight %}

After calling `#.compile_assets`, calls to assets in your views will default to a using a single link 
each to your CSS and Javascript compiled asset files.  

By default the compiled files are written to the public directory, so that they can be served by the webserver.


### Asset Compression

If you have the `yuicompressor` gem installed and working, it will be used automatically to compress 
your Javascript and CSS assets.  

Otherwise, the assets will just be concatenated together and not compressed during compilation.


### With Asset Groups

When using asset groups, a separate compiled file will be produced per asset group.


### Unique Asset Names

When compiling assets, a unique name is given to each asset file, using the a `SHA1` hash of the 
content of the file.  This is done so that clients do not attempt to use cached versions of the assets 
if the asset has changed.


### Serving

If you call `r.assets` when compiling assets, then the app will serve the compiled asset files.  

However, it is recommended to have the main webserver (e.g. nginx) serve the compiled files, instead 
of relying on the application.

Assuming you are using compiled assets in production mode that are served by the webserver, you can 
remove the serving of them by the application:


{% highlight ruby %}
route do |r|
  r.assets unless ENV['RACK_ENV'] == 'production'
end
{% endhighlight %}


If you do have the application serve the compiled assets, it will use the `Last-Modified` header to 
make sure that clients do not re-download compiled assets that haven't changed.


### Asset Pre-Compilation

If you want to precompile your assets, so they do not need to be compiled every time you boot the 
application, you can provide a `:precompiled` option when loading the plugin.  

The value of this option should be the filename where the compiled asset metadata is stored.  

If the compiled asset metadata file does not exist when the **:assets** plugin is loaded, the plugin will 
run in non-compiled mode.  

However, when you call `#.compile_assets`, it will write the compiled asset metadata file after compiling the assets.

If the compiled asset metadata file already exists when the assets plugin is loaded, the plugin will 
read the file to get the compiled asset metadata, and it will run in compiled mode, assuming that the 
compiled asset files already exist.


### On Heroku

[Heroku](https://heroku.com/) supports precompiling the assets when using Roda.  

You just need to add an `assets:precompile` task, similar to this:

{% highlight ruby %}
namespace :assets do
  
  desc "Precompile the assets"
  task :precompile do
    require './app'
    App.compile_assets
  end
  
end
{% endhighlight %}



## Plugin Options

| **Option:** | **Description:** |
| --- | --- |
| :add_suffix                     | Whether to append a `.css` or `.js` extension to asset routes in non-compiled mode (default: `false`) |
| :compiled_css_dir               | Directory name in which to store the compiled CSS file, inside `:compiled_path` (default: `nil`) |
| :compiled_css_route             | Route under `:prefix` for compiled CSS assets (default: `:compiled_css_dir`) |
| :compiled_js_dir                | Directory name in which to store the compiled Javascript file, inside :compiled_path (default: `nil`) |
| :compiled_js_route              | Route under `:prefix` for compiled Javascript assets (default: `:compiled_js_dir`) |
| :compiled_name                  | Compiled file name prefix (default: `'app'`) |
| :compiled_path                  | Path inside public folder in which compiled files are stored (default: `:prefix`) |
| :concat_only                    | Whether to just concatenate instead of concatenating and compressing files (default: `false`) |
| :css_dir                        | Directory name containing your CSS source, inside `:path` (default: `'css'`) |
| :css_headers                    | A hash of additional headers for your rendered CSS files. |
| :css_opts                       | Template options to pass to the render plugin (via `:template_opts`) when rendering CSS assets |
| :css_route                      | Route under `:prefix` for CSS assets (default: `:css_dir`)
| :dependencies                   | A hash of dependencies for your asset files.  Keys should be paths to asset files, values should be arrays of paths your asset files depends on.  This is used to detect changes in your asset files. |
| :group_subdirs                  | Whether a hash used in `:css` and `:js` options requires the assets for the related group are contained in a subdirectory with the same name (default: `true`) |
| :headers                        | A hash of additional headers for both JS and CSS rendered files |
| :js_dir                         | Directory name containing your Javascript source, inside :path (default: 'js') |
| :js_headers                     | A hash of additional headers for your rendered Javascript files
| :js_opts                        | Template options to pass to the render plugin (via `:template_opts`) when rendering Javascript assets
| :js_route                       | Route under `:prefix` for Javascript assets (default: :js_dir)
| :path                           | Path to your asset source directory (default: `'assets'`).  Relative paths will be considered relative to the application's `:root` option. |
| :prefix                         | Prefix for assets path in your URL/routes (default: `'assets'`) |
| :precompiled                    | Path to the compiled asset metadata file.  If the file exists, will use compiled mode using the metadata in the file.  If the file does not exist, will use non-compiled mode, but will write the metadata to the file if `#.compile_assets` is called. |
| :public                         | Path to your public folder, in which compiled files are placed (default: `'public'`). Relative paths will be considered relative to the application's `:root` option. |



### ClassMethods

Return the assets options for this class.

`#.assets_opts`


Compile options for the given asset type. If no `asset_type` is given, compiles both the `:css` and `:js` asset types.  

You can specify an array of types (e.g. `[:css, :frontend]`) to compile assets for the given asset group.

{% highlight ruby %}
compile_assets(type=nil)
{% endhighlight %}


### InstanceMethods

#### `#.assets(type, attrs = nil)`

Return a string containing html tags for the given asset type. This will use a script tag for the `:js` 
type and a link tag for the `:css` type.

To return the tags for a specific asset group, use an array for the type, such as `[:css, :frontend]`.

When the assets are not compiled, this will result in a separate tag for each asset file.  When the 
assets are compiled, this will result in a single tag to the compiled asset file.


#### `#.render_asset(file, type)`

Render the asset with the given filename.  When assets are compiled, or when the file is already of 
the given type (no rendering necessary), this returns the contents of the compiled file.

When assets are not compiled and the file is not already of the correct, this will render the asset 
using the render plugin. 

In both cases, if the file has not been modified since the last request, this will return a `304` response.



#### `#.read_asset_file(file, type)`

Return the content of the file if it is already of the correct type.

Otherwise, render the file using the render plugin. `file` should be the relative path to the file 
from the current directory.



### RequestClassMethods

An array of asset type strings and regexps for that type, for all asset types handled.

`#.assets_matchers`



### RequestMethods


Render the matching asset if this is a GET request for a supported asset.

{% highlight ruby %}
r.assets
{% endhighlight %}


