---
title: [Ruby, Roda - Web Framework, Plugins]
layout: default
asset\_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

<div id="toc"></div>

---

## Introduction

This page should contain a quick overview of available Roda plugins.


---
## Plugins - <span>(built-in)</span>

### Core Routing

| **Plugin Name:** | **Description:** |
| --- | --- |
| <h4>[:all_verbs](all_verbs.html)</h4>                      | Adds request routing methods for all http verbs.|
| <h4>[:backtracking_array](backtracking_array.html)</h4>    | Allows array matchers to backtrack if later matchers do not match. |
| <h4>[:class_level_routing](class_level_routing.html)</h4>  | Adds class level routing methods, for a DSL similar to [Sinatra](https://github.com/sinatra/sinatra). |
| <h4>[:head](head.html)</h4>                                | Treat HEAD requests like GET requests with an empty response body. |
| <h4>[:multi_route](multi_route.html)</h4>                  | Allows for multiple named route blocks that can be dispatched to inside main route block. |
| <h4>[:multi_run](multi_run.html)</h4>                      | Adds the ability to dispatch to multiple rack applications based on the request path prefix. |
| <h4>[:not_allowed](not_allowed.html)</h4>                  | Adds support for automatically returning 405 Method Not Allowed responses. |
| <h4>[:run_handler](run_handler.html)</h4>                  | Allows for modifying rack response arrays when using r.run, and continuing routing for 404 responses. |


### Rendering

| **Plugin Name:** | **Description:** |
| --- | --- |
| <h4>[:assets](assets.html)</h4>                             | Adds support for rendering CSS/JS javascript assets on the fly in development, or compiling them into a single compressed file in production. |
| <h4>[:chunked](chunked.html)</h4>                           | Adds support for streaming template responses using Transfer-Encoding: chunked. |
| <h4>[:json](json.html)</h4>                                 | Allows match blocks to return arrays and hashes, using a json representation as the response body. |
| <h4>[:named_templates](named_templates.html)</h4>           | Adds the ability to create inline templates by name, instead of storing them in the file system. |
| <h4>[:padrino_render](padrino_render.html)</h4>             | Makes render method that work similarly to Padrino's rendering, using a layout by default. |
| <h4>[:partials](partials.html)</h4>                         | Adds partial method for rendering partials (templates prefixed with an underscore). |
| <h4>[:precompile_templates](precompile_templates.html)</h4> | Adds support for precompiling templates, saving memory when using a forking webserver. |
| <h4>[:render](render.html)</h4>                             | Adds render method for rendering templates, using [tilt](https://github.com/rtomayko/tilt). |
| <h4>[:static](static.html)</h4>                             | Adds support for serving static files using Rack::Static. |
| <h4>[:streaming](streaming.html)</h4>                       | Adds ability to stream responses. |
| <h4>[:symbol_views](symbol_views.html)</h4>                 | Allows match blocks to return template name symbols, uses the template view as the response body. |
| <h4>[:view_options](view_options.html)</h4>                 | Allows for setting view options on a per-request basis. |
| <h4>[:websockets](websockets.html)</h4>                     | Adds websocket support using [faye-websocket](https://github.com/faye/faye-websocket-ruby"). |


### View Helpers

| **Plugin Name:** | **Description:** |
| --- | --- |
| <h4>[:content_for](content_for.html)</h4>                   | Allows storage of content in one template and retrieval of that content in a different template. |
| <h4>[:csrf](csrf.html)</h4>                                 | Adds CSRF protection and helper methods using [rack\_csrf](https://github.com/baldowl/rack_csrf). |
| <h4>[:h](h.html)</h4>                                       | Adds `h` method for html escaping. |
| <h4>[:render_each](render_each.html)</h4>                   | Render a template for each value in an enumerable. |


### Request/Response Helpers

| **Plugin Name:** | **Description:** |
| --- | --- |
| <h4>[:caching](caching.html)</h4>                           | Adds request and response methods related to http caching. |
| <h4>[:cookies](cookies.html)</h4>                           | Adds response methods for handling cookies. |
| <h4>[:default_headers](default_headers.html)</h4>           | Allows modifying the default headers for responses. |
| <h4>[:default_status](default_status.html)</h4>             | Allows overriding the default status for responses. |
| <h4>[:delegate](delegate.html)</h4>                         | Adds class methods for creating instance methods that delegate to the request, response, or class. |
| <h4>[:delete_empty_headers](delete_empty_headers.html)</h4> | Automatically delete response headers with empty values. |
| <h4>[:drop_body](drop_body.html)</h4>                       | Automatically drops response body and Content-Type/Content-Length headers for response statuses indicating no body. |
| <h4>[:halt](halt.html)</h4>                                 | Augments request halt method for support for setting response status and/or response body. |
| <h4>[:module_include](module_include.html)</h4>             | Adds request_module and response_module class methods for adding modules/methods to request/response classes. |
| <h4>[:response_request](response_request.html)</h4>         | Gives response object access to request object. |
| <h4>[:sinatra_helpers](sinatra_helpers.html)</h4>           | Port of Sinatra::Helpers methods not covered by other plugins. |


### Routing Helpers

| **Plugin Name:** | **Description:** |
| --- | --- |
| <h4>[:error_handler](error_handler.html)</h4>               | Adds ability to automatically handle errors raised by the application. |
| <h4>[:hooks](hooks.html)</h4>                               | Adds before/after hook methods. |
| <h4>[:not_found](not_found.html)</h4>                       | Adds not_found method for handling responses not otherwise handled by a route. |
| <h4>[:pass](pass.html)</h4>                                 | Adds pass method for skipping the current matching route block as if it didn't match. |
| <h4>[:path_rewriter](path_rewriter.html)</h4>               | Adds support for rewriting paths before routing. |
| <h4>[:status_handler](status_handler.html)</h4>             | Adds status_handler method for handling responses without bodies for a given status code. |


### Matchers

| **Plugin Name:** | **Description:** |
| --- | --- |
| <h4>[:empty_root](empty_root.html)</h4>                     | Makes root matcher match empty string in addition to single slash. |
| <h4>[:hash_matcher](hash_matcher.html)</h4>                 | Adds hash_matcher class method for easily defining hash matchers. |
| <h4>[:header_matchers](header_matchers.html)</h4>           | Adds matchers using information from the request headers. |
| <h4>[:match_affix](match_affix.html)</h4>                   | Adds support for overriding default prefix/suffix used in match patterns. |
| <h4>[:param_matchers](param_matchers.html)</h4>             | Adds matchers using information from the request params. |
| <h4>[:path_matchers](path_matchers.html)</h4>               | Adds matchers using information from the request path. |
| <h4>[:slash_path_empty](slash_path_empty.html)</h4>         | Considers a path of &quot;/&quot; as an empty path when doing a terminal match. |
| <h4>[:symbol_matchers](symbol_matchers.html)</h4>           | Adds support for symbol-specific matching regexps. |


### Other

| **Plugin Name:** | **Description:** |
| --- | --- |
| <h4>[:delay_build](delay_build.html)</h4>                   | Delay building the rack app until Roda.app is called. |
| <h4>[:environments](environments.html)</h4>                 | Adds support for handling different execution environments (development/test/production). |
| <h4>[:error_email](error_email.html)</h4>                   | Adds ability to easily email a notification when an error is raised by the application. |
| <h4>[:flash](flash.html)</h4>                               | Adds flash handling. |
| <h4>[:heartbeat](heartbeat.html)</h4>                       | Adds support for heartbeats. |
| <h4>[:indifferent_params](indifferent_params.html)</h4>     | Adds params method for indifferent parameters. |
| <h4>[:json_parser](json_parser.html)</h4>                   | Parses request bodies in JSON format. |
| <h4>[:mailer](mailer.html)</h4>                             | Adds support for sending emails using the routing tree. |
| <h4>[:middleware](middleware.html)</h4>                     | Allows the Roda app to be used as middleware by another app. |
| <h4>[:path](path.html)</h4>                                 | Adds support for named paths. |
| <h4>[:per_thread_caching](per_thread_caching.html)</h4>     | Switches the thread-safe cache from a shared cache to a per-thread cache. |
| <h4>[:shared_vars](shared_vars.html)</h4>                   | Stores and retrieves variables shared between multiple Roda apps. |

<br>


---
## Plugins - <span>(external)</span>

| **Plugin Name:** | **Description:** |
| --- | --- |
| <h4>[:autoforme](https://github.com/jeremyevans/autoforme)</h4>                       | Adds autoforme method for automatic creation of administrative front-end for Sequel models. |
| <h4>[:forme](https://github.com/jeremyevans/forme)</h4>                               | Adds form method for simple creation of html forms inside erb templates. |
| <h4>[:rodauth](https://github.com/jeremyevans/rodauth)</h4>                           | Authentication Framework for Roda/Sequel/PostgreSQL. |
| <h4>[:roda-action](https://github.com/AMHOL/roda-action)</h4>                         | Resolves actions stored in roda-container. |
| <h4>[:roda-auth](https://github.com/beno/roda-auth)</h4>                              | Adds authentication support for Roda. |
| <h4>[:roda-component](https://github.com/cj/roda-component)</h4>                      | Adds realtime components using faye and opal. |
| <h4>[:roda-container](https://github.com/AMHOL/roda-container)</h4>                   | Turns application into an inversion of control (IoC) container. |
| <h4>[:roda-flow](https://github.com/AMHOL/roda-flow)</h4>                             | Changes routing methods to delegate to containers. |
| <h4>[:roda-i18n](https://github.com/kematzy/roda-i18n)</h4>                           | Adds easy internationalization and localization support. |
| <h4>[:roda-parse-request](https://github.com/3scale/roda-parse-request)</h4>          | Automatically parse JSON and URL-encoded requests. |
| <h4>[:roda-route_list](https://github.com/jeremyevans/roda-route_list)</h4>           | Parses route metadata from comments in an app file, allowing introspection of routes. |
| <h4>[:roda-rest_api](https://github.com/beno/roda-rest_api)</h4>                      | Adds support for easily creating RESTful APIs. |
| <h4>[:roda-symbolized_params](https://github.com/janko-m/roda-symbolized_params)</h4> | Adds params method for symbolized params. |
| <h4>[:roda-will_paginate](https://github.com/manuca/roda-will_paginate)</h4>          | `:will_paginate` integration for Roda. |
| <h4>[:rom-roda](https://github.com/rom-rb/rom-roda)</h4>                              | Adds integration with Ruby Object Mapper. |



