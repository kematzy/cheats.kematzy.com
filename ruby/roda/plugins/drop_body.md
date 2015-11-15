---
title: [Ruby, Roda - Web Framework, Plugins, ":drop_body"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The drop_body plugin automatically drops the body and `Content-Type`/`Content-Length` headers from the 
response if the response status indicates that the response should not include a body 
(response statuses `100`, `101`, `102`, `204`, `205`, and `304`).


### ResponseMethods


#### `#.finish`

If the response status indicates a body should not be returned, use an empty body and remove the 
`Content-Length` and `Content-Type` headers.
