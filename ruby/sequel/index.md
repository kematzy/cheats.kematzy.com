---
title: [Ruby, Sequel ORM]
layout: default
asset_path: ../..

---

# {{ page.title | join: " / " }}

----

<div id="toc"></div>

---

## Sub-pages

### [Validations](/ruby/sequel/validations.html)

### [Plugins](/ruby/sequel/plugins.html)

### [Models](/ruby/sequel/models.html)

### [Associations](/ruby/sequel/associations.html)

### [Datasets](/ruby/sequel/datasets.html)

### [Migrations](/ruby/sequel/migrations.html)



----

## Overview

This page should contain an overview of Sequel's key features as well as links to all other parts.


<pre><code class="ruby sql">
 # 1. Load Sequel
require "sequel"

 # 2. Add global extensions
Sequel.extension(:blank)

 # 3. Connnect to DB
configure :development do
  DB = Sequel.connect(:adapter=>'mysql2', :host=>'localhost', :database=>'rubymodularapp-dev-development', :user=>'root', :password=>'root')
end

configure :production do
   # DB = Sequel.connect(:adapter=>'mysql', :host=>'localhost', :database=>'ppt-db-app', :user=>'root', :password=>'root')
   DB = Sequel.sqlite('db/production.db')
 end

configure :test do
  DB = Sequel.sqlite # :memory:
  # DB = Sequel.sqlite('db/test.db')
end

 # 4. Add Global Sequel plugins, each loaded separately
Sequel::Model.plugin(:schema)
Sequel::Model.plugin(:validation_helpers)
Sequel::Model.plugin(:timestamps)
 # Add JSON output capability to all model instances
Sequel::Model.plugin(:json_serializer)
 # Add XML output capability to all model instances
Sequel::Model.plugin(:xml_serializer)
Sequel::Model.plugin(:paranoid)

 # 5. Load Model Classes
require_relative 'app/models'
</code></pre>




### CRUD


---
## DB Creations

---


---



### Open a database

{% highlight ruby %}  
require 'rubygems'
require 'sequel'

DB = Sequel.sqlite('my_blog.db')
DB = Sequel.connect('postgres://user:password@localhost/my_db')
DB = Sequel.postgres('my_db', :user => 'user', :password => 'password', :host => 'localhost')
DB = Sequel.ado('mydb')
{% endhighlight %}

### Open an SQLite memory database

Without a filename argument, the sqlite adapter will setup a new sqlite database in memory.

{% highlight ruby %}
DB = Sequel.sqlite
{% endhighlight %}

### Logging SQL statements

{% highlight ruby %}  
require 'logger'
DB = Sequel.sqlite '', :loggers => [Logger.new($stdout)]
 # or
DB.loggers << Logger.new(...)
{% endhighlight %}

### Using raw SQL

{% highlight ruby %}  
DB.run "CREATE TABLE users (name VARCHAR(255) NOT NULL, age INT(3) NOT NULL)"
dataset = DB["SELECT age FROM users WHERE name = ?", name]
dataset.map(:age)
DB.fetch("SELECT name FROM users") do |row|
  p row[:name]
end
{% endhighlight %}

### Create a dataset

{% highlight ruby %}  
dataset = DB[:items]
dataset = DB.from(:items)
{% endhighlight %}

### Most dataset methods are chainable

{% highlight ruby %}  
dataset = DB[:managers].where(:salary => 5000..10000).order(:name, :department)
{% endhighlight %}

### Insert rows

{% highlight ruby %}  
dataset.insert(:name => 'Sharon', :grade => 50)
{% endhighlight %}

### Retrieve rows

{% highlight ruby %}  
dataset.each{|r| p r}
dataset.all # => [{...}, {...}, ...]
dataset.first # => {...}
{% endhighlight %}

### Update/Delete rows

{% highlight ruby %}  
dataset.filter(~:active).delete
dataset.filter('price < ?', 100).update(:active => true)
{% endhighlight %}

### Datasets are Enumerable

{% highlight ruby %}  
dataset.map{|r| r[:name]}
dataset.map(:name) # same as above

dataset.inject(0){|sum, r| sum + r[:value]}
dataset.sum(:value) # same as above
{% endhighlight %}

### Filtering (see also doc/dataset_filtering.rdoc)

#### Equality

{% highlight ruby %}  
dataset.filter(:name => 'abc')
dataset.filter('name = ?', 'abc')
{% endhighlight %}

#### Inequality

{% highlight ruby %}  
dataset.filter{value > 100}
dataset.exclude{value <= 100}
{% endhighlight %}

#### Inclusion

{% highlight ruby %}  
dataset.filter(:value => 50..100)
dataset.where{(value >= 50) & (value <= 100)}

dataset.where('value IN ?', [50,75,100])
dataset.where(:value=>[50,75,100])

dataset.where(:id=>other_dataset.select(:other_id))
{% endhighlight %}

#### Subselects as scalar values

{% highlight ruby %}  
dataset.where('price > (SELECT avg(price) + 100 FROM table)')
dataset.filter{price > dataset.select(avg(price) + 100)}
{% endhighlight %}

#### LIKE/Regexp

{% highlight ruby %}  
DB[:items].filter(:name.like('AL%'))
DB[:items].filter(:name => /^AL/)
{% endhighlight %}

#### AND/OR/NOT

{% highlight ruby %}  
DB[:items].filter{(x > 5) & (y > 10)}.sql 
  # SELECT * FROM items WHERE ((x > 5) AND (y > 10))

DB[:items].filter({:x => 1, :y => 2}.sql_or & ~{:z => 3}).sql 
  # SELECT * FROM items WHERE (((x = 1) OR (y = 2)) AND (z != 3))
{% endhighlight %}

#### Mathematical operators

{% highlight ruby %}  
DB[:items].filter((:x + :y) > :z).sql 
  # SELECT * FROM items WHERE ((x + y) > z)

DB[:items].filter{price - 100 < avg(price)}.sql 
  # SELECT * FROM items WHERE ((price - 100) < avg(price))
{% endhighlight %}

### Ordering

{% highlight ruby %}  
dataset.order(:kind)
dataset.reverse_order(:kind)
dataset.order(:kind.desc, :name)
{% endhighlight %}

### Limit/Offset

{% highlight ruby %}  
dataset.limit(30) # LIMIT 30
dataset.limit(30, 10) # LIMIT 30 OFFSET 10
{% endhighlight %}

### Joins

{% highlight ruby %}  
DB[:items].left_outer_join(:categories, :id => :category_id).sql 
  # SELECT * FROM items LEFT OUTER JOIN categories ON categories.id = items.category_id

DB[:items].join(:categories, :id => :category_id).join(:groups, :id => :items__group_id) 
  # SELECT * FROM items INNER JOIN categories ON categories.id = items.category_id INNER JOIN groups ON groups.id = items.group_id
{% endhighlight %}

### Aggregate functions methods

{% highlight ruby %}  
dataset.count #=> record count
dataset.max(:price)
dataset.min(:price)
dataset.avg(:price)
dataset.sum(:stock)

dataset.group_and_count(:category)
dataset.group(:category).select(:category, :AVG.sql_function(:price))
{% endhighlight %}

### SQL Functions / Literals

{% highlight ruby %}  
dataset.update(:updated_at => :NOW.sql_function)
dataset.update(:updated_at => 'NOW()'.lit)

dataset.update(:updated_at => "DateValue('1/1/2001')".lit)
dataset.update(:updated_at => :DateValue.sql_function('1/1/2001'))
{% endhighlight %}

### Schema Manipulation

{% highlight ruby %}  
DB.create_table :items do
  primary_key :id
  String :name, :unique => true, :null => false
  TrueClass :active, :default => true
  foreign_key :category_id, :categories
  DateTime :created_at
  
  index :created_at
end

DB.drop_table :items

DB.create_table :test do
  String :zipcode
  enum :system, :elements => ['mac', 'linux', 'windows']
end
{% endhighlight %}

### Aliasing 

{% highlight ruby %}  
DB[:items].select(:name.as(:item_name))
DB[:items].select(:name___item_name)
DB[:items__items_table].select(:items_table__name___item_name)
  # SELECT items_table.name AS item_name FROM items AS items_table
{% endhighlight %}


## Transactions

{% highlight ruby %}  
DB.transaction do
  dataset.insert(:first_name => 'Inigo', :last_name => 'Montoya')
  dataset.insert(:first_name => 'Farm', :last_name => 'Boy')
end # Either both are inserted or neither are inserted
{% endhighlight %}


Database#transaction is re-entrant:


{% highlight ruby %}  
DB.transaction do # BEGIN issued only here
  DB.transaction
    dataset << {:first_name => 'Inigo', :last_name => 'Montoya'}
  end
end # COMMIT issued only here
{% endhighlight %}


Transactions are aborted if an error is raised:


{% highlight ruby %}  
DB.transaction do
  raise "some error occurred"
end # ROLLBACK issued and the error is re-raised
{% endhighlight %}

Transactions can also be aborted by raising Sequel::Rollback:

{% highlight ruby %}  
DB.transaction do
  raise(Sequel::Rollback) if something_bad_happened
end # ROLLBACK issued and no error raised
{% endhighlight %}

Savepoints can be used if the database supports it:


{% highlight ruby %}  
DB.transaction do
  dataset << {:first_name => 'Farm', :last_name => 'Boy'} # Inserted
  DB.transaction(:savepoint=>true) # This savepoint is rolled back
    dataset << {:first_name => 'Inigo', :last_name => 'Montoya'} # Not inserted
    raise(Sequel::Rollback) if something_bad_happened
  end
  dataset << {:first_name => 'Prince', :last_name => 'Humperdink'} # Inserted
end
{% endhighlight %}

### Miscellaneous:

{% highlight ruby %}  
dataset.sql # "SELECT * FROM items"
dataset.delete_sql # "DELETE FROM items"
dataset.where(:name => 'sequel').exists # "EXISTS ( SELECT * FROM items WHERE name = 'sequel' )"
dataset.columns #=> array of columns in the result set, does a SELECT
DB.schema(:items) => [[:id, {:type=>:integer, ...}], [:name, {:type=>:string, ...}], ...]
{% endhighlight %}

---

### Documents

{% highlight ruby %}  
http://sequel.rubyforge.org/rdoc/files/doc/association_basics_rdoc.html
http://sequel.rubyforge.org/rdoc/classes/Sequel/Schema/Generator.html
http://sequel.rubyforge.org/rdoc/files/doc/validations_rdoc.html
http://sequel.rubyforge.org/rdoc/classes/Sequel/Model.html
{% endhighlight %}

### Alter table

{% highlight ruby %}  
database.alter_table :deals do
  add_column :name, String
  drop_column :column_name
  rename_column :from, :to

  add_constraint :valid_name, :name.like('A%')
  drop_constraint :constraint

  add_full_text_index :body
  add_spacial_index [columns]

  add_index :price
  drop_index :index

  add_foreign_key :artist_id, :table
  add_primary_key :id
  add_unique_constraint [columns]
  set_column_allow_null :foo, false
  set_column_default :title, ''

  set_column_type :price, 'char(10)'
end
{% endhighlight %}



### Model stuff


#### instance.delete

Deletes and returns `self`. **Does not run destroy hooks**. Look into using destroy instead.

{% highlight ruby %}  
Artist[1].delete # DELETE FROM artists WHERE (id = 1)
 # => #<Artist {:id=>1, ...}>
{% endhighlight %}  

{% highlight ruby %}  
deal = Deal[1]

deal.changed_columns

deal.destroy  # Calls hooks
deal.delete   # No hooks

deal.exists?

deal.new?  # returns true if new

deal.hash  # Only uniques

deal.keys  #=> [:id, :name]

deal.modified!  # set modified? to true, even if no changes
deal.modified?  # true if modified

deal.lock!
{% endhighlight %}

### Callbacks

{% highlight ruby %}  
before_create
after_create

before_validation
after_validation

before_save
before_update
  UPDATE QUERY
after_update
after_save

before_destroy
  DELETE QUERY
after_destroy
{% endhighlight %}

### Schema

{% highlight ruby %}  
class Deal < Sequel::Model
  set_schema do
    primary_key :id
    primary_key [:id, :title]  # composite primary key
    String :name, primary_key: true
    
    String  :title
    Numeric :price
    DateTime :expires

    unique :whatever
    check(:price) { num > 0 }

    foreign_key :artist_id
    String :artist_name, key: :id

    index :title
    index [:artist_id, :name]
    full_text_index :title

    # String, Integer, Fixnum, Bignum, Float, Numeric, BigDecimal,
    # Date, DateTime, Time, File, TrueClass, FalseClass
  end
end
{% endhighlight %}

### Unrestrict primary key

{% highlight ruby %}  
Category.create(id: 'travel')   # error
Category.unrestrict_primary_key
Category.create(id: 'travel')   # ok
{% endhighlight %}