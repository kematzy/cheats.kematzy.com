---
title: [Ruby, Sequel ORM, Models]
layout: default
asset_path: ../..

---

# {{ page.title | join: " / " }}

----

`
Sequel::Model` is an object relational mapper built on top of Sequel core. Each model class is backed by a dataset instance, and many dataset methods can be called directly on the class. Model datasets return rows as model instances, which have fairly standard ORM instance behaviour.

`Sequel::Model` is built completely out of plugins. 

Plugins can override any class, instance, or dataset method defined by a previous plugin and call super to get the default behaviour. 

By default, `Sequel::Model` loads two plugins, Sequel::Model (which is itself a plugin) for the base support, and `Sequel::Model::Associations` for the associations support.

You can set the `SEQUEL_NO_ASSOCIATIONS` constant or environment variable to make Sequel not load the associations plugin by default.

## Classes and Modules

## Sequel::Model::Associations
## Sequel::Model::ClassMethods
## Sequel::Model::DatasetMethods
## Sequel::Model::InstanceMethods
## Sequel::Model::DatasetModule
## Sequel::Model::Errors


### Constants

| **Sequel::Model Constants** |||
| --- | --- | --- |
| AFTER_HOOKS | =	[:after_create, :after_update, :after_save, :after_destroy, :after_validation, :after_commit, :after_rollback, :after_destroy_commit, :after_destroy_rollback] | Hooks that are called after an action. When overriding these, it is recommended to call super on the first line of your method, so later hooks are called after earlier hooks.
| ANONYMOUS_MODEL_CLASSES | =	{} | Map that stores model classes created with Sequel::Model(), to allow the reopening of classes when dealing with code reloading.
| ANONYMOUS_MODEL_CLASSES_MUTEX | =	Mutex.new | Mutex protecting access to ANONYMOUS_MODEL_CLASSES
| AROUND_HOOKS | = [:around_create, :around_update, :around_save, :around_destroy, :around_validation] | Hooks that are called around an action. If overridden, these methods must call super exactly once if the behaviour they wrap is desired. The can be used to rescue exceptions raised by the code they wrap or ensure that some behaviour is executed no matter what.
| BEFORE_HOOKS | = [:before_create, :before_update, :before_save, :before_destroy, :before_validation] | Hooks that are called before an action. Can return false to not do the action. When overriding these, it is recommended to call super as the last line of your method, so later hooks are called before earlier hooks.
| BOOLEAN_SETTINGS | = [:typecast_empty_string_to_nil, :typecast_on_assignment, :strict_param_setting, \ :raise_on_save_failure, :raise_on_typecast_failure, :require_modification, :use_after_commit_rollback, :use_transactions] | Boolean settings that can be modified at the global, class, or instance level.
| DATASET_METHODS | = (Dataset::ACTION_METHODS + Dataset::QUERY_METHODS + [:each_server]) - [:and, :or, :[], :columns, :columns!, :delete, :update, :add_graph_aliases, :first, :first!] | Class methods added to model that call the method of the same name on the dataset
| HOOKS | = BEFORE_HOOKS + AFTER_HOOKS | Empty instance methods to create that the user can override to get hook/callback behaviour. Just like any other method defined by Sequel, if you override one of these, you should call super to get the default behaviour (while empty by default, they can also be defined by plugins). See the “Model Hooks” guide for more detail on hooks.
| INHERITED_INSTANCE_VARIABLES | = {:@allowed_columns=>:dup, :@dataset_method_modules=>:dup, :@primary_key=>nil, :@use_transactions=>nil, :@raise_on_save_failure=>nil, :@require_modification=>nil, :@restricted_columns=>:dup, :@restrict_primary_key=>nil, :@simple_pk=>nil, :@simple_table=>nil, :@strict_param_setting=>nil, :@typecast_empty_string_to_nil=>nil, :@typecast_on_assignment=>nil, :@raise_on_typecast_failure=>nil, :@plugins=>:dup, :@setter_methods=>nil, :@use_after_commit_rollback=>nil, :@fast_pk_lookup_sql=>nil, :@fast_instance_delete_sql=>nil, :@finders=>:dup, :@finder_loaders=>:dup, :@db=>nil, :@default_set_fields_options=>:dup} | Class instance variables that are inherited in subclasses. If the value is :dup, dup is called on the superclass's instance variable when creating the instance variable in the subclass. If the value is nil, the superclass's instance variable is used directly in the subclass.
| NORMAL_METHOD_NAME_REGEXP | = /\A[A-Za-z_][A-Za-z0-9_]*\z/ | Regular expression that determines if a method name is normal in the sense that it could be used literally in ruby code without using send. Used to avoid problems when using eval with a string to define methods.
| OPTS | = Sequel::OPTS | RESTRICTED_SETTER_METHODS	=	instance_methods.map{|x| x.to_s}.grep(SETTER_METHOD_REGEXP) | The setter methods (methods ending with =) that are never allowed to be called automatically via set/update/new/etc..
| SETTER_METHOD_REGEXP | = /=\z/ | Regular expression that determines if the method is a valid setter name (i.e. it ends with =).


## Instance Methods

`Sequel::Model` instance methods that implement basic model functionality.

All of the methods in `HOOKS` and `AROUND_HOOKS` create instance methods that are called by Sequel when the appropriate action occurs. For example, when destroying a model object, Sequel will call `around_destroy`, which will call `before_destroy`, do the destroy, and then call `after_destroy`.

The following instance_methods all call the class method of the same name: `columns`, `db`, `primary_key`, `db_schema`.

All of the methods in `BOOLEAN_SETTINGS` create `attr_writers` allowing you to set values for the attribute. It also creates instance getters returning the value of the setting. If the value has not yet been set, it gets the default value from the class by calling the class method of the same name.


### Attributes

| **Attributes** ||
| --- | --- | --- |
| to_hash | [R] | The hash of attribute values. Keys are symbols with the names of the underlying database columns.

Artist.new(:name=>'Bob').values # => {:name=>'Bob'}
Artist[1].values # => {:id=>1, :name=>'Jim', ...}

| values | [R] | The hash of attribute values. Keys are symbols with the names of the underlying database columns.

Artist.new(:name=>'Bob').values # => {:name=>'Bob'}
Artist[1].values # => {:id=>1, :name=>'Jim', ...}

## Public Class methods


<hr class="divider">
### new (values = {})

Creates new instance and passes the given values to set. If a block is given, yield the instance to the block unless from_db is true.

| Arguments:||
| --- | --- |
| values | should be a hash to pass to set.
| from_db | only for backwards compatibility, forget it exists.

{% highlight ruby %}
Artist.new(:name=>'Bob')

Artist.new do |a|
  a.name = 'Bob'
end
{% endhighlight %}


## Public Instance methods

<hr class="divider">
### == (obj)

Alias of `eql?`


<hr class="divider">
### === (obj)

If `pk` is not `nil`, true only if the objects have the same class and pk. If pk is nil, false.

{% highlight ruby %}
Artist[1] === Artist[1] # true
Artist.new === Artist.new # false
Artist[1].set(:name=>'Bob') == Artist[1] # => true
{% endhighlight %}


<hr class="divider">
### [] (column)

Returns value of the column's attribute.

{% highlight ruby %}
Artist[1][:id] #=> 1
{% endhighlight %}


<hr class="divider">
### []= (column, value)

Sets the value for the given column. If typecasting is enabled for this object, typecast the value based on the column's type. If this is a new record or the typecasted value isn't the same as the current value for the column, mark the column as changed.

{% highlight ruby %}
a = Artist.new
a[:name] = 'Bob'
a.values              #=> {:name=>'Bob'}
{% endhighlight %}


<hr class="divider">
### autoincrementing_primary_key ()

The autoincrementing primary key for this model object. Should be overridden if you have a composite primary key with one part of it being autoincrementing.

<hr class="divider">
### changed_columns ()

The columns that have been updated. This isn't completely accurate, as it could contain columns whose values have not changed.

{% highlight ruby %}
a = Artist[1]
a.changed_columns # => []
a.name = 'Bob'
a.changed_columns # => [:name]
{% endhighlight %}

<hr class="divider">
### delete ()

Deletes and returns self. Does not run destroy hooks. Look into using destroy instead.

{% highlight ruby %}
Artist[1].delete      #=> DELETE FROM artists WHERE (id = 1)
 # => #<Artist {:id=>1, ...}>
{% endhighlight %}


<hr class="divider">
### destroy (opts = OPTS)

Like delete but runs hooks before and after delete. If `before_destroy` returns false, returns false without deleting the object from the database. Otherwise, deletes the item from the database and returns self. Uses a transaction if `use_transactions` is true or if the `:transaction` option is given and true.

{% highlight ruby %}
Artist[1].destroy         #=> BEGIN; DELETE FROM artists WHERE (id = 1); COMMIT;
 # => #<Artist {:id=>1, ...}>
{% endhighlight %}


<hr class="divider">
### each (&block)

Iterates through all of the current values using each.

Album[1].each{|k, v| puts "#{k} => #{v}"}
 # id => 1
 # name => 'Bob'


<hr class="divider">
### eql? (obj)

Compares model instances by values.

{% highlight ruby %}
Artist[1] == Artist[1]                    # => true
Artist.new == Artist.new                  # => true
Artist[1].set(:name=>'Bob') == Artist[1]  # => false
{% endhighlight %}


<hr class="divider">
### errors ()
Returns the validation errors associated with this object. See Errors.


<hr class="divider">
### exists? ()

Returns true when current instance exists, false otherwise. Generally an object that isn't new will exist unless it has been deleted. Uses a database query to check for existence, unless the model object is new, in which case this is always false.

{% highlight ruby %}
Artist[1].exists?       #=> SELECT 1 FROM artists WHERE (id = 1)
 # => true
Artist.new.exists?
 # => false
{% endhighlight %}


### extend (mod)

Ignore the model's setter method cache when this instances extends a module, as the module may contain setter methods.

<hr class="divider">
### freeze ()

Freeze the object in such a way that it is still usable but not modifiable. Once an object is frozen, you cannot modify it's values, #changed_columns, errors, or dataset.


<hr class="divider">
### hash ()

Value that should be unique for objects with the same class and pk (if pk is not nil), or the same class and values (if pk is nil).

{% highlight ruby %}
Artist[1].hash == Artist[1].hash # true
Artist[1].set(:name=>'Bob').hash == Artist[1].hash # true
Artist.new.hash == Artist.new.hash # true
Artist.new(:name=>'Bob').hash == Artist.new.hash # false
{% endhighlight %}


<hr class="divider">
### id ()

Returns value for the :id attribute, even if the primary key is not id. To get the primary key value, use pk.

{% highlight ruby %}
Artist[1].id # => 1
{% endhighlight %}


<hr class="divider">
### inspect ()

Returns a string representation of the model instance including the class name and values.


<hr class="divider">
### keys ()

Returns the keys in values. May not include all column names.

{% highlight ruby %}
Artist.new.keys                 # => []
Artist.new(:name=>'Bob').keys   # => [:name]
Artist[1].keys                  # => [:id, :name]
{% endhighlight %}


<hr class="divider">
### lock! ()

Refresh this record using for_update unless this is a new record. Returns self. This can be used to make sure no other process is updating the record at the same time.

{% highlight ruby %}
a = Artist[1]
Artist.db.transaction do
  a.lock!
  a.update(:name=>'A')
end
{% endhighlight %}


<hr class="divider">
### marshallable! ()

Remove elements of the model object that make marshalling fail. Returns self.

{% highlight ruby %}
a = Artist[1]
a.marshallable!
Marshal.dump(a)
{% endhighlight %}


<hr class="divider">
### modified! (column=nil)

Explicitly mark the object as modified, so save_changes/update will run callbacks even if no columns have changed.

{% highlight ruby %}
a = Artist[1]
a.save_changes # No callbacks run, as no changes
a.modified!
a.save_changes # Callbacks run, even though no changes made
{% endhighlight %}

If a column is given, specifically marked that column as modified, so that save_changes/update will include that column in the update. This should be used if you plan on mutating the column value instead of assigning a new column value:

{% highlight ruby %}
a.modified!(:name)
a.name.gsub!(/[aeou]/, 'i')
{% endhighlight %}


<hr class="divider">
### modified? (column=nil)

Whether this object has been modified since last saved, used by #save_changes to determine whether changes should be saved. New values are always considered modified.

{% highlight ruby %}
a = Artist[1]
a.modified? # => false
a.set(:name=>'Jim')
a.modified? # => true
{% endhighlight %}

If a column is given, specifically check if the given column has been modified:

{% highlight ruby %}
a.modified?(:num_albums) # => false
a.num_albums = 10
a.modified?(:num_albums) # => true
{% endhighlight %}


<hr class="divider">
### new? ()

Returns true if the current instance represents a new record.

{% highlight ruby %}
Artist.new.new? # => true
Artist[1].new? # => false
{% endhighlight %}


<hr class="divider">
### pk ()

Returns the primary key value identifying the model instance. Raises an Error if this model does not have a primary key. If the model has a composite primary key, returns an array of values.

{% highlight ruby %}
Artist[1].pk # => 1
Artist[[1, 2]].pk # => [1, 2]
{% endhighlight %}


<hr class="divider">
### pk_hash ()

Returns a hash mapping the receivers primary key column(s) to their values.

{% highlight ruby %}
Artist[1].pk_hash           # => {:id=>1}
Artist[[1, 2]].pk_hash      # => {:id1=>1, :id2=>2}
{% endhighlight %}


<hr class="divider">
### qualified_pk_hash (qualifier=model.table_name)

Returns a hash mapping the receivers primary key column(s) to their values.

{% highlight ruby %}
Artist[1].qualified_pk_hash         # => {Sequel.qualify(:artists, :id)=>1}
Artist[[1, 2]].qualified_pk_hash    # => {Sequel.qualify(:artists, :id1)=>1, Sequel.qualify(:artists, :id2)=>2}
{% endhighlight %}


<hr class="divider">
### refresh ()

Reloads attributes from database and returns self. Also clears all #changed_columns information. Raises an Error if the record no longer exists in the database.

{% highlight ruby %}
a = Artist[1]
a.name = 'Jim'
a.refresh
a.name # => 'Bob'
{% endhighlight %}


<hr class="divider">
### reload ()

Alias of refresh, but not aliased directly to make overriding in a plugin easier.

<hr class="divider">
### save (opts=OPTS)

Creates or updates the record, after making sure the record is valid and before hooks execute successfully. Fails if:

* the record is not valid, or

* before_save returns false, or

* the record is new and before_create returns false, or

* the record is not new and before_update returns false.


If save fails and either `raise_on_save_failure` or the `:raise_on_failure` option is true, it raises `ValidationFailed` or `HookFailed`. Otherwise it returns `nil`.

If it succeeds, it returns self.

You can provide an optional list of columns to update, in which case it only updates those columns, or a options hash.


| Takes the following options: ||
| --- | --- |
| :changed | save all changed columns, instead of all columns or the columns given
| :columns | array of specific columns that should be saved.
| :raise_on_failure	| set to true or false to override the current raise_on_save_failure setting
| :server	| set the server/shard on the object before saving, and use that server/shard in any transaction.
| :transaction	| set to true or false to override the current use_transactions setting
| :validate	| set to false to skip validation


<hr class="divider">
### save_changes (opts=OPTS)

Saves only changed columns if the object has been modified. If the object has not been modified, returns nil. If unable to save, returns false unless raise_on_save_failure is true.

{% highlight ruby %}
a = Artist[1]
a.save_changes # => nil
a.name = 'Jim'
a.save_changes # UPDATE artists SET name = 'Bob' WHERE (id = 1)
 # => #<Artist {:id=>1, :name=>'Jim', ...}
{% endhighlight %}


<hr class="divider">
### set (hash)

Updates the instance with the supplied values with support for virtual attributes, raising an exception if a value is used that doesn't have a setter method (or ignoring it if strict_param_setting = false). Does not save the record.

{% highlight ruby %}
artist.set(:name=>'Jim')
artist.name   # => 'Jim'
{% endhighlight %}


<hr class="divider">
### set_all (hash)

Set all values using the entries in the hash, ignoring any setting of allowed_columns in the model.

{% highlight ruby %}
Artist.set_allowed_columns(:num_albums)
artist.set_all(:name=>'Jim')
artist.name         # => 'Jim'
{% endhighlight %}


<hr class="divider">
### set_fields (hash, fields, opts=nil)

For each of the fields in the given array fields, call the setter method with the value of that hash entry for the field. Returns self.

| You can provide an options hash, with the following options currently respected: ||
| --- | --- |
| :missing | Can be set to :skip to skip missing entries or :raise to raise an Error for missing entries. The default behavior is not to check for missing entries, in which case the default value is used. To be friendly with most web frameworks, the missing check will also check for the string version of the argument in the hash if given a symbol.


**Examples:**

{% highlight ruby %}
artist.set_fields({:name=>'Jim'}, [:name])
artist.name # => 'Jim'

artist.set_fields({:hometown=>'LA'}, [:name])
artist.name # => nil
artist.hometown # => 'Sac'

artist.name # => 'Jim'
artist.set_fields({}, [:name], :missing=>:skip)
artist.name # => 'Jim'

artist.name # => 'Jim'
artist.set_fields({}, [:name], :missing=>:raise)
 # Sequel::Error raised

{% endhighlight %}


<hr class="divider">
### set_only (hash, *only)

Set the values using the entries in the hash, only if the key is included in only. It may be a better idea to use set_fields instead of this method.

{% highlight ruby %}
artist.set_only({:name=>'Jim'}, :name)
artist.name # => 'Jim'

artist.set_only({:hometown=>'LA'}, :name)     # Raise Error
{% endhighlight %}


<hr class="divider">
### set_server (s)

Set the shard that this object is tied to. Returns self.


<hr class="divider">
### singleton_method_added (meth)

Clear the setter_methods cache when a method is added


<hr class="divider">
### this ()

Returns (naked) dataset that should return only this instance.

{% highlight ruby %}
Artist[1].this                #=> SELECT * FROM artists WHERE (id = 1) LIMIT 1
{% endhighlight %}


<hr class="divider">
### update (hash)

Runs set with the passed hash and then runs save_changes.

{% highlight ruby %}
artist.update(:name=>'Jim')      #=> UPDATE artists SET name = 'Jim' WHERE (id = 1)
{% endhighlight %}


<hr class="divider">
### update_all (hash)

Update all values using the entries in the hash, ignoring any setting of allowed_columns in the model.

{% highlight ruby %}
Artist.set_allowed_columns(:num_albums)
artist.update_all(:name=>'Jim')           #=> UPDATE artists SET name = 'Jim' WHERE (id = 1)
{% endhighlight %}


<hr class="divider">
### update_fields (hash, fields, opts=nil)

Update the instances values by calling set_fields with the arguments, then saves any changes to the record. Returns self.

{% highlight ruby %}
artist.update_fields({:name=>'Jim'}, [:name])       #=> UPDATE artists SET name = 'Jim' WHERE (id = 1)

artist.update_fields({:hometown=>'LA'}, [:name])    #=> UPDATE artists SET name = NULL WHERE (id = 1)
{% endhighlight %}


<hr class="divider">
### update_only (hash, *only)

Update the values using the entries in the hash, only if the key is included in only. It may be a better idea to use update_fields instead of this method.

{% highlight ruby %}
artist.update_only({:name=>'Jim'}, :name)   #=> UPDATE artists SET name = 'Jim' WHERE (id = 1)

artist.update_only({:hometown=>'LA'}, :name) # Raise Error
{% endhighlight %}


<hr class="divider">
### valid? (opts = OPTS)

Validates the object and returns true if no errors are reported.

{% highlight ruby %}
artist(:name=>'Valid').valid?       # => true
artist(:name=>'Invalid').valid?     # => false
artist.errors.full_messages         # => ['name cannot be Invalid']
{% endhighlight %}


<hr class="divider">
### validate ()

Validates the object. If the object is invalid, errors should be added to the errors attribute. By default, does nothing, as all models are valid by default. See the “Model Validations” guide. for details about validation. Should not be called directly by user code, call valid? instead to check if an object is valid.


--- 

## Class Methods

Class methods for Sequel::Model that implement basic model functionality.

All of the method names in Model::DATASET_METHODS have class methods created that call the Model's dataset with the method of the same name with the given arguments.

### Constants

{% highlight ruby %}
FINDER_TYPES  =  [:first, :all, :each, :get].freeze     
{% endhighlight %}


| **Attributes** |||
| --- | --- | --- |
| allowed_columns | [R] | Which columns should be the only columns allowed in a call to a mass assignment method (e.g. set) (default: not set, so all columns not otherwise restricted are allowed).
| dataset_method_modules | [R] | Array of modules that extend this model's dataset. Stored so that if the model's dataset is changed, it will be extended with all of these modules.
| default_set_fields_options | [RW]	| The default options to use for Model#set_fields. These are merged with the options given to set_fields.
| fast_instance_delete_sql | [R] | SQL string fragment used for faster DELETE statement creation when deleting/destroying model instances, or nil if the optimization should not be used. For internal use only.
| instance_dataset | [R] | The dataset that instance datasets (#this) are based on. Generally a naked version of the model's dataset limited to one row. For internal use only.
| plugins | [R]	 | Array of plugin modules loaded by this class <br>{% highlight ruby %}Sequel::Model.plugins # => [Sequel::Model, Sequel::Model::Associations]{% endhighlight %}

| **Attributes** |||
| --- | --- | --- |
| primary_key | [R] | The primary key for the class. Sequel can determine this automatically for many databases, but not all, so you may need to set it manually. If not determined automatically, the default is :id.
| raise_on_save_failure | [RW] | Whether to raise an error instead of returning nil on a failure to save/create/save_changes/update/destroy due to a validation failure or a before_* hook returning false (default: true).
| raise_on_typecast_failure | [RW] | Whether to raise an error when unable to typecast data for a column (default: true). This should be set to false if you want to use validations to display nice error messages to the user (e.g. most web applications). You can use the validates_schema_types validation (from the validation_helpers plugin) in connection with this setting to check for typecast failures during validation.
| require_modification | [RW] | Whether to raise an error if an UPDATE or DELETE query related to a model instance does not modify exactly 1 row. If set to false, Sequel will not check the number of rows modified (default: true).
| simple_pk | [R] | Should be the literal primary key column name if this Model's table has a simple primary key, or nil if the model has a compound primary key or no primary key.
| simple_table | [R] | Should be the literal table name if this Model's dataset is a simple table (no select, order, join, etc.), or nil otherwise. This and #simple_pk are used for an optimization in Model.[].
| strict_param_setting | [RW] | Whether new/set/update and their variants should raise an error if an invalid key is used. A key is invalid if no setter method exists for that key or the access to the setter method is restricted (e.g. due to it being a primary key field). If set to false, silently skip any key where the setter method doesn't exist or access to it is restricted.
| typecast_empty_string_to_nil | [RW] | Whether to typecast the empty string ('') to nil for columns that are not string or blob. In most cases the empty string would be the way to specify a NULL SQL value in string form (nil.to_s == ''), and an empty string would not usually be typecast correctly for other types, so the default is true.
| typecast_on_assignment | [RW] | Whether to typecast attribute values on assignment (default: true). If set to false, no typecasting is done, so it will be left up to the database to typecast the value correctly.
| use_after_commit_rollback | [RW] | Whether to enable the after_commit and after_rollback hooks when saving/destroying instances. On by default, can be turned off for performance reasons or when using prepared transactions (which aren't compatible with after commit/rollback).
| use_transactions | [RW] | Whether to use a transaction by default when saving/deleting records (default: true). If you are sending database queries in before_* or after_* hooks, you shouldn't change the default setting without a good reason.


## Public Instance methods

<hr class="divider">
### [] (*args)

Returns the first record from the database matching the conditions. If a hash is given, it is used as the conditions. If another object is given, it finds the first record whose primary key(s) match the given argument(s). If no object is returned by the dataset, returns nil.

{% highlight ruby %}
Artist[1] # SELECT * FROM artists WHERE id = 1
 # => #<Artist {:id=>1, ...}>

Artist[:name=>'Bob'] # SELECT * FROM artists WHERE (name = 'Bob') LIMIT 1
 # => #<Artist {:name=>'Bob', ...}>
{% endhighlight %}

<hr class="divider">
### call (values)

Initialises a model instance as an existing record. This constructor is used by Sequel to initialise model instances when fetching records. Requires that values be a hash where all keys are symbols. It probably should not be used by external code.

<hr class="divider">
### clear_setter_methods_cache ()

Clear the #setter_methods cache

<hr class="divider">
### columns ()

Returns the columns in the result set in their original order. Generally, this will use the columns determined via the database schema, but in certain cases (e.g. models that are based on a joined dataset) it will use Dataset#columns to find the columns.

{% highlight ruby %}
Artist.columns  # => [:id, :name]
{% endhighlight %}


<hr class="divider">
### create (values = {}, &block)

Creates instance using new with the given values and block, and saves it.

{% highlight ruby %}
Artist.create(:name=>'Bob')
 # INSERT INTO artists (name) VALUES ('Bob')

Artist.create do |a|
  a.name = 'Jim'
end # INSERT INTO artists (name) VALUES ('Jim')
{% endhighlight %}


<hr class="divider">
### dataset ()

Returns the dataset associated with the Model class. Raises an Error if there is no associated dataset for this class. In most cases, you don't need to call this directly, as Model proxies many dataset methods to the underlying dataset.

{% highlight ruby %}
Artist.dataset.all # SELECT * FROM artists
{% endhighlight %}

<hr class="divider">
### dataset= (ds)

Alias of `#set_dataset`


<hr class="divider">
### dataset_module (mod = nil)

Extend the dataset with a module, similar to adding a plugin with the methods defined in DatasetMethods. This is the recommended way to add methods to model datasets.

If an argument, it should be a module, and is used to extend the underlying dataset. Otherwise an anonymous module is created, and if a block is given, it is module_evaled, allowing you do define dataset methods directly using the standard ruby def syntax. Returns the module given or the anonymous module created.

{% highlight ruby %}
 # Usage with existing module
Artist.dataset_module Sequel::ColumnsIntrospection

 # Usage with anonymous module
Artist.dataset_module do
  def foo
    :bar
  end
end
Artist.dataset.foo    # => :bar
Artist.foo            # => :bar
{% endhighlight %}


Any anonymous modules created are actually instances of Sequel::Model::DatasetModule (a Module subclass), which allows you to call the subset method on them:

{% highlight ruby %}
Artist.dataset_module do
  subset :released, Sequel.identifier(release_date) > Sequel::CURRENT_DATE
end
{% endhighlight %}

Any public methods in the dataset module will have class methods created that call the method on the dataset, assuming that the class method is not already defined.

<hr class="divider">
### db ()

Returns the database associated with the Model class. 

If this model doesn't have a database associated with it, assumes the superclass's database, or the first object in `Sequel::DATABASES`. 

If no Sequel::Database object has been created, raises an error.

{% highlight ruby %}
Artist.db.transaction do # BEGIN
  Artist.create(:name=>'Bob')     #=> INSERT INTO artists (name) VALUES ('Bob')
end # COMMIT
{% endhighlight %}


<hr class="divider">
### db= (db)

Sets the database associated with the Model class. 

If the model has an associated dataset, sets the model's dataset to a dataset on the new database with the same options used by the current dataset. 

This can be used directly on Sequel::Model to set the default database to be used by subclasses, or to override the database used for specific models:

{% highlight ruby %}
Sequel::Model.db = DB1
Artist.db = DB2
{% endhighlight %}

Note that you should not use this to change the model's database at runtime. If you have that need, you should look into Sequel's sharding support.


<hr class="divider">
### db_schema ()

Returns the cached schema information if available or gets it from the database. This is a hash where keys are column symbols and values are hashes of information related to the column. See Database#schema.

{% highlight ruby %}
Artist.db_schema
 # {:id => { :type=>:integer, :primary_key=>true, ... },
 #  :name => { :type=>:string, :primary_key=>false, ...} }
{% endhighlight %}

<hr class="divider">
### def_column_alias (meth, column)

Create a column alias, where the column methods have one name, but the underlying storage uses a different name.

<hr class="divider">
### def_dataset_method (*args, &block)

If a block is given, define a method on the dataset (if the model currently has an dataset) with the given argument name using the given block. Also define a class method on the model that calls the dataset method. Stores the method name and block so that it can be reapplied if the model's dataset changes.

If a block is not given, just define a class method on the model for each argument that calls the dataset method of the same argument name.

It is recommended that you define methods inside a block passed to dataset_module instead of using this method, as `dataset_module` allows you to use normal ruby def syntax.

{% highlight ruby %}
 # Add new dataset method and class method that calls it
Artist.def_dataset_method(:by_name){order(:name)}
Artist.filter(:name.like('A%')).by_name
Artist.by_name.filter(:name.like('A%'))

 # Just add a class method that calls an existing dataset method
Artist.def_dataset_method(:server!)
Artist.server!(:server1)
{% endhighlight %}


<hr class="divider">
### find (*args, &block)

Finds a single record according to the supplied filter. You are encouraged to use Model.[] or Model.first instead of this method.

{% highlight ruby %}
Artist.find(:name=>'Bob')     #=> SELECT * FROM artists WHERE (name = 'Bob') LIMIT 1

Artist.find { name > 'M' }    #=> SELECT * FROM artists WHERE (name > 'M') LIMIT 1

{% endhighlight %}

<hr class="divider">
### find_or_create (cond, &block)

Like find but invokes create with given conditions when record does not exist. Unlike find in that the block used in this method is not passed to find, but instead is passed to create only if find does not return an object.

{% highlight ruby %}
Artist.find_or_create(:name=>'Bob')
 # SELECT * FROM artists WHERE (name = 'Bob') LIMIT 1
 # INSERT INTO artists (name) VALUES ('Bob')

Artist.find_or_create(:name=>'Jim'){|a| a.hometown = 'Sactown'}
 # SELECT * FROM artists WHERE (name = 'Jim') LIMIT 1
 # INSERT INTO artists (name, hometown) VALUES ('Jim', 'Sactown')
{% endhighlight %}



<hr class="divider">
### finder (meth=OPTS, opts=OPTS, &block)

Create an optimised finder method using a dataset placeholder literalizer. This pre-computes the SQL to use for the query, except for given arguments.

There are two ways to use this. The recommended way is to pass a symbol that represents a model class method that returns a dataset:

{% highlight ruby %}
def Artist.by_name(name)
  where(:name=>name)
end

Artist.finder :by_name
{% endhighlight %}

This creates an optimised first_by_name method, which you can call normally:

{% highlight ruby %}
Artist.first_by_name("Joe")
{% endhighlight %}


The alternative way to use this to pass your own block:

{% highlight ruby %}
Artist.finder(:name=>:first_by_name) { |pl, ds| ds.where(:name=>pl.arg).limit(1) }
{% endhighlight %}

**Note!! if you pass your own block, you are responsible for manually setting limits if necessary (as shown above**).

| **Options:** ||
| --- | --- |
| :arity	| When using a symbol method name, this specifies the arity of the method. This should be used if if the method accepts an arbitrary number of arguments, or the method has default argument values. Note that if the method is defined as a dataset method, the class method Sequel creates accepts an arbitrary number of arguments, so you should use this option in that case. If you want to handle multiple possible arities, you need to call the finder method multiple times with unique :arity and :name methods each time.
| :name	| The name of the method to create. This must be given if you pass a block. If you use a symbol, this defaults to the symbol prefixed by the type.
| :mod | The module in which to create the finder method. Defaults to the singleton class of the model.
| :type	| The type of query to run. Can be :first, :each, :all, or :get, defaults to :first.
| Caveats: | This doesn't handle all possible cases. For example, if you have a method such as:


{% highlight ruby %}
def Artist.by_name(name)
  name ? where(:name=>name) : exclude(:name=>nil)
end
{% endhighlight %}

Then calling a finder without an argument will not work as you expect.

{% highlight ruby %}
Artist.finder :by_name
Artist.by_name(nil).first       #=> WHERE (name IS NOT NULL)
Artist.first_by_name(nil)       #=> WHERE (name IS NULL)
{% endhighlight %}

See `Dataset::PlaceholderLiteralizer` for additional caveats.


<hr class="divider">
### first (*args, &block)

An alias for calling first on the model's dataset, but with optimized handling of the single argument case.


<hr class="divider">
### first! (*args, &block)

An alias for calling first! on the model's dataset, but with optimized handling of the single argument case.


<hr class="divider">
### implicit_table_name ()

Returns the implicit table name for the model class, which is the demodulized, underscored, pluralized name of the class.

{% highlight ruby %}
Artist.implicit_table_name              # => :artists
Foo::ArtistAlias.implicit_table_name    # => :artist_aliases
{% endhighlight %}


<hr class="divider">
### include (*mods)

Clear the #setter_methods cache when a module is included, as it may contain setter methods.


<hr class="divider">
### inherited (subclass)

If possible, set the dataset for the model subclass as soon as it is created. Also, make sure the inherited class instance variables are copied into the subclass.

Sequel queries the database to get schema information as soon as a model class is created:

{% highlight ruby %}
class Artist < Sequel::Model # Causes schema query
end
{% endhighlight %}


<hr class="divider">
### load (values)

Calls call with the values hash. Only for backwards compatibility.


<hr class="divider">
### method_added (meth)

Clear the #setter_methods cache when a setter method is added


<hr class="divider">
### no_primary_key ()

Mark the model as not having a primary key. Not having a primary key can cause issues, among which is that you won't be able to update records.

{% highlight ruby %}
Artist.primary_key        # => :id
Artist.no_primary_key
Artist.primary_key        # => nil
{% endhighlight %}


<hr class="divider">
### plugin (plugin, *args, &block)

Loads a plugin for use with the model class, passing optional arguments to the plugin. 

If the plugin is a module, load it directly. Otherwise, require the plugin from either sequel/plugins/#{plugin} or sequel_#{plugin}, and then attempt to load the module using a the camelized plugin name under Sequel::Plugins.


<hr class="divider">
### prepared_finder (meth=OPTS, opts=OPTS, &block)

Similar to finder, but uses a prepared statement instead of a placeholder literalizer. This makes the SQL used static (cannot vary per call), but allows binding argument values instead of literalizing them into the SQL query string.

If a block is used with this method, it is `instance_execed` by the model, and should accept the desired number of placeholder arguments.

The options are the same as the options for finder, with the following exception: 

| ** OPTS:** ||
| --- | --- |
| :type	| Specifies the type of prepared statement to create


<hr class="divider">
### primary_key_hash (value)

Returns primary key attribute hash. If using a composite primary key value such be an array with values for each primary key in the correct order. For a standard primary key, value should be an object with a compatible type for the key. If the model does not have a primary key, raises an Error.

{% highlight ruby %}
Artist.primary_key_hash(1) # => {:id=>1}
Artist.primary_key_hash([1, 2]) # => {:id1=>1, :id2=>2}
{% endhighlight %}


<hr class="divider">
### qualified_primary_key_hash (value, qualifier=table_name)

Return a hash where the keys are qualified column references. Uses the given qualifier if provided, or the #table_name otherwise. This is useful if you plan to join other tables to this table and you want the column references to be qualified.

{% highlight ruby %}
Artist.filter(Artist.qualified_primary_key_hash(1))
 # SELECT * FROM artists WHERE (artists.id = 1)
{% endhighlight %}


<hr class="divider">
### restrict_primary_key ()

Restrict the setting of the primary key(s) when using mass assignment (e.g. set). Because this is the default, this only make sense to use in a subclass where the parent class has used `unrestrict_primary_key`.

<hr class="divider">
### restrict_primary_key? ()

Whether or not setting the primary key(s) when using mass assignment (e.g. set) is restricted, true by default.

<hr class="divider">
### set_allowed_columns (*cols)

Set the columns to allow when using mass assignment (e.g. set). Using this means that any columns not listed here will not be modified. If you have any virtual setter methods (methods that end in =) that you want to be used during mass assignment, they need to be listed here as well (without the =).

It may be better to use a method such as set_only or set_fields that lets you specify the allowed fields per call.

{% highlight ruby %}
Artist.set_allowed_columns(:name, :hometown)
Artist.set(:name=>'Bob', :hometown=>'Sactown') # No Error
Artist.set(:name=>'Bob', :records_sold=>30000) # Error
{% endhighlight %}

<hr class="divider">
### set_dataset (ds, opts=OPTS)

Sets the dataset associated with the Model class. ds can be a `Symbol`, `LiteralString`, `SQL::Identifier`, `SQL::QualifiedIdentifier`, `SQL::AliasedExpression` (all specifying a table name in the current database), or a Dataset. If a dataset is used, the model's database is changed to the database of the given dataset. If a dataset is not used, a dataset is created from the current database with the table name given. Other arguments raise an Error. Returns self.

This changes the row_proc of the dataset to return model objects and extends the dataset with the dataset_method_modules. It also attempts to determine the database schema for the model, based on the given dataset.

{% highlight ruby %}
Artist.set_dataset(:tbl_artists)
Artist.set_dataset(DB[:artists])
{% endhighlight %}

Note that you should not use this to change the model's dataset at runtime. If you have that need, you should look into Sequel's sharding support.


<hr class="divider">
### set_primary_key (key)

Sets the primary key for this model. You can use either a regular or a composite primary key. To not use a primary key, set to nil or use no_primary_key. On most adapters, Sequel can automatically determine the primary key to use, so this method is not needed often.

{% highlight ruby %}
class Person < Sequel::Model
  # regular key
  set_primary_key :person_id
end

class Tagging < Sequel::Model
  # composite key
  set_primary_key [:taggable_id, :tag_id]
end
{% endhighlight %}


<hr class="divider">
### setter_methods ()

Cache of setter methods to allow by default, in order to speed up new/set/update instance methods.

<hr class="divider">
### subset (name, *args, &block)

Sets up a dataset method that returns a filtered dataset. Sometimes thought of as a scope, and like most dataset methods, they can be chained. For example:

{% highlight ruby %}
Topic.subset(:joes, :username.like('%joe%'))
Topic.subset(:popular){num_posts > 100}
Topic.subset(:recent){created_on > Date.today - 7}
{% endhighlight %}

Allows you to do:

{% highlight ruby %}
Topic.joes.recent.popular
{% endhighlight %}

to get topics with a username that includes joe that have more than 100 posts and were created less than 7 days ago.

Both the args given and the block are passed to `Dataset#filter`.

This method creates dataset methods that do not accept arguments. To create dataset methods that accept arguments, you should use define a method directly inside a dataset_module block.

<hr class="divider">
### table_name ()

Returns name of primary table for the dataset. If the table for the dataset is aliased, returns the aliased name.

{% highlight ruby %}
Artist.table_name                     # => :artists
Sequel::Model(:foo).table_name        # => :foo
Sequel::Model(:foo___bar).table_name  # => :bar
{% endhighlight %}


<hr class="divider">
### unrestrict_primary_key ()

Allow the setting of the primary key(s) when using the mass assignment methods. Using this method can open up security issues, be very careful before using it.

{% highlight ruby %}
Artist.set(:id=>1)      # Error
Artist.unrestrict_primary_key
Artist.set(:id=>1)      # No Error
{% endhighlight %}

<hr class="divider">
### with_pk (pk)

Return the model instance with the primary key, or nil if there is no matching record.

<hr class="divider">
### with_pk! (pk)

Return the model instance with the primary key, or raise `NoMatchingRow` if there is no matching record.



