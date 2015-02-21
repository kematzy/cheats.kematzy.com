---
title: [Ruby, Sequel ORM, Associations]
layout: default
asset_path: ../..

---

# {{ page.title | join: " / " }}

----

<div id="toc"></div>

----

## Introduction


--- 

## Model Associations


### one_to_one <small>- short description goes here....</small>

<hr class="divider">

### one_to_many <small>- short description goes here....</small>

<hr class="divider">

### many_to_one <small>- short description goes here....</small>

<hr class="divider">


### many_to_many <small>- short description goes here....</small>


--- 

## Model Associations

{% highlight ruby %}  
class Deal < Sequel::Model

  # Us (left) <=> Them (right)
  many_to_many  :images,
    left_id:    :deal_id,
    right_id:   :image_id,
    join_table: :image_links

  one_to_many   :files,
    key:        :deal_id,
    class:      :DataFile,

  many_to_one   :parent, class: self
  one_to_many   :children, key: :parent_id, class: self

  one_to_many :gold_albums, class: :Album do |ds|
    ds.filter { copies_sold > 50000 }
  end
{% endhighlight %}

Provided by many_to_many

{% highlight ruby %}  
Deal[1].images
Deal[1].add_image
Deal[1].remove_image
Deal[1].remove_all_images
{% endhighlight %}


[Associations Basics](http://sequel.rubyforge.org/rdoc/files/doc/association_basics_rdoc.html)