---
title: [Ruby, Roda - Web Framework, Plugins, ":slash_path_empty"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:slash\_path\_empty** plugin considers `"/"` as an empty path, in addition to the default 
of `""` being considered an empty path.  

This makes it so `r.is` without an argument will match a path of `"/"`, and `r.is` and verb methods 
such as `r.get` and `r.post` will match if the path is `"/"` after the arguments are processed.  

This can make it easier to handle applications where a trailing `"/"` in the path should be ignored.

### TODO: add more examples

