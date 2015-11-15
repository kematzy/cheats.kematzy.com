---
title: [Ruby, Roda - Web Framework, Plugins, ":delay_build"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:delay_build** plugin does not build the rack app until `Roda.app` is called, and only rebuilds 
the Rack app if `Roda.build!` is called.  

This differs from Roda's default behaviour, which rebuilds the Rack app every time the route block 
changes and every time middleware is added if a route block has already been defined.

If you are loading hundreds of middleware after a route block has already been defined, this can fix 
a possible performance issue, turning an `O(n^2)` calculation into an `O(n)` calculation, where `n` 
is the number of middleware used.


### ClassMethods

#### `#.app`

If the app is not been defined yet, build the app.


#### `#.build!`

Rebuild the application.


