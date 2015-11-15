---
title: [Ruby, Roda - Web Framework, Plugins, ":delete_empty_headers"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction

The **:delete_empty_headers** plugin deletes any headers whose value is set to the empty string.  

Because of how default headers are set in `Roda`, if you have a default header but don't want
to set it for a specific request, you need to use this plugin and set the header value to the empty 
string, and Roda will automatically delete the header.


### TODO: add some valid examples




### ResponseMethods

#### `#.finish`

Delete any empty headers when calling finish


#### `#,def finish_with_body(_)`

Delete any empty headers when calling finish_with_body

