---
title: Plugins - Sequel ORM
layout: default
asset_path: ../..

---

# Ruby / Sequel ORM / Plugins

{% highlight ruby %} 
{% endhighlight %}


---

## Schema

{% highlight ruby %}
 # affect all models
Sequel::Model.plugin(:schema)

 # only one model
class Post < Sequel::Model
  plugin :schema
end
{% endhighlight %}

---

## ValidationHelpers

{% highlight ruby %}
 # affect all models
Sequel::Model.plugin(:validation_helpers)

 # only one model
class Post < Sequel::Model
  plugin :validation_helpers
end
{% endhighlight %}

---

## Timestamp

{% highlight ruby %}
 # affect all models
Sequel::Model.plugin(:timestamps)

 # only one model
class Post < Sequel::Model
  plugin :timestamps
end
{% endhighlight %}


---

## List

---

## Tree


---

## [Paranoid](https://github.com/kematzy/sequel-paranoid)

A plugin for the Ruby ORM Sequel, that allows soft deletion of database entries.

### Basics

In order to use the paranoid plugin in the very basic version, just add it your model like this:

{% highlight ruby %}
 # affect all models
Sequel::Model.plugin(:paranoid)

 # affects only one model
class Post < Sequel::Model
  plugin :paranoid
end
{% endhighlight %}


This will assume that you have a column `deleted_at`, which gets filled with the current timestamp once the model gets destroyed:

{% highlight ruby %}
instance = ParanoidModel.create(:something)

instance.deleted?   # => false
instance.deleted_at # => nil

instance.destroy

instance.deleted?   # => true
instance.deleted_at # => current timestamp

{% endhighlight %}

### Reading the data

By default the plugin will not change the way scopes have been working. 

So if you want to take the deletion state of an entry into account you can use the following dataset filters:

{% highlight ruby %}
ParanoidModel.present.all      # => Will return all the non-deleted entries from the db.
ParanoidModel.deleted.all      # => Will return all the deleted entries from the db.
ParanoidModel.with_deleted.all # => Will ignore the deletion state (and is the default).
{% endhighlight %}


### Renaming the deletion timestamp columns

If you don't want to use the default column name `deleted_at`, you can easily rename that column:

{% highlight ruby %}
class ParanoidModel < Sequel::Model
  plugin :paranoid, :deleted_at_field_name => :destroyed_at
end

instance = ParanoidModel.create(:something => 'foo')

instance.destroy
instance.destroyed_at # => current timestamp
{% endhighlight %}


### Enabling the non-deleted default scope

In order to exclude deleted entries by default from any query, you can enable an option in the plugin. One major reason for don't enabling it by default is the fact, that associations are kinda broken, when you want to load also deleted associated instances:

{% highlight ruby %}
class ParanoidModel < Sequel::Model
  plugin :paranoid, :enable_default_scope => true
  one_to_many :child_models
end

class AnotherModel < Sequel::Model
  plugin :paranoid
  many_to_one :paranoid_model
end

 # create some dummy data

parent1 = ParanoidModel.create(:something => 'foo')
parent2 = ParanoidModel.create(:something => 'bar')

child1  = ChildModel.create(:something => 'foo')
child2  = ChildModel.create(:something => 'bar')
child3  = ChildModel.create(:something => 'baz')

parent1.add_child_model(child1)
parent1.add_child_model(child2)
parent2.add_child_model(child3)

 # destroy one of the children

child1.destroy
child1.deleted? # => true

 # load the children

ChildModel.all                              # => [child2, child3] (works as expected)
ChildModel.dataset.unfiltered.all           # => [child1, child2, child3] (works as expected)

parent1.child_models_dataset.all            # => [child2] (works as expected)
parent1.child_models_dataset.unfiltered.all # => [child1, child2, child3] (broken)

{% endhighlight %}


Note that **the last command is broken**, as `child3` is not associated with `parent1`. The reason for that is `unfiltered`, which will not only remove the `deleted_at` check but also the association condition of the query.



---

## JSON Serializer

Add JSON output capability to all model instances

{% highlight ruby %}
 # affect all models
Sequel::Model.plugin(:json_serializer)

 # affects only one model
class Post < Sequel::Model
  plugin :json_serializer
end
{% endhighlight %}


---

## XML Serializer

Add XML output capability to all model instances

{% highlight ruby %}
 # affect all models
Sequel::Model.plugin(:xml_serializer)

 # only one model
class Post < Sequel::Model
  plugin :xml_serializer
end
{% endhighlight %}

---
