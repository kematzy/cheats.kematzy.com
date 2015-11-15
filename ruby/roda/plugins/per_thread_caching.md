---
title: [Ruby, Roda - Web Framework, Plugins, ":per_thread_caching"]
layout: default
asset_path: ../../..

---

# {{ page.title | join: " / " }}

---- 

## Introduction


The **:per_thread_caching** plugin changes the default `cache` from being a shared thread safe `cache` 
to a separate cache per thread.  

This means getting or setting values no longer needs a mutex on non-MRI ruby implementations, which 
may be faster when using a thread pool.  However, since the caches are no longer shared, this will take up more memory.

**Note!** It does not make sense to use this plugin on `MRI`, since the default cache on `MRI` doesn't 
use a mutex as it is already thread safe due to the GVL.

Using this plugin changes the matcher regexp cache to use per-thread caches, and changes the default 
for future thread-safe caches to use per-thread caches.

**NOTE!**

If you want the **:render** plugin's template cache to use per-thread caches, you should load this 
plugin before the **:render** plugin.


--- 

### ClassMethods

Use the per-thread cache instead of the default cache.
#### `#.thread_safe_cache`
