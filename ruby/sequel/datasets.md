---
title: [Ruby, Sequel ORM, Datasets]
layout: default
asset_path: ../..

---

# {{ page.title | join: " / " }}

----

<div id="toc"></div>

---

## Introduction 

Datasets are the primary way Sequel uses to access the database. 

While most database libraries have specific support for updating all records or only a single record, Sequel's ability to represent SQL queries themselves as objects is what gives Sequel most of its power. 

However, if you haven't been exposed to the dataset concept before, it can be a little disorienting. This document aims to give a basic introduction to datasets and how to use them.

### What a Dataset Represents

A Dataset can be thought of representing one of two concepts:

* An SQL query

* An abstract set of rows and some related behaviour

The first concept is more easily understood, so you should probably start with that assumption.

## Basics

The most basic dataset is the simple selection of all columns in a table:

{% highlight ruby %}
ds = DB[:posts]   # SELECT * FROM posts
{% endhighlight %}

Here, DB represents your `Sequel::Database` object, and `ds` is your dataset, with the SQL query it represents below it.

One of the core dataset ideas that should be understood is that datasets use a functional style of modification, in which methods called on the dataset return modified copies of the dataset, they don't modify the dataset themselves:

{% highlight ruby %}
ds2 = ds.where(:id=>1)

ds2  #=> SELECT * FROM posts WHERE id = 1
ds #=> SELECT * FROM posts
{% endhighlight %}


Note how `ds` itself **is not modified**. This is because `ds.where` **returns a modified copy of `ds`**, instead of modifying `ds` itself. This makes **using datasets both thread safe and easy to chain**:

{% highlight ruby %}
 # Thread safe:
100.times do |i|
  Thread.new do
    ds.where(:id=>i).first
  end
end

 # Easy to chain:
ds3 = ds.select(:id, :name).order(:name).where{id < 100}   #=> SELECT id, name FROM posts WHERE id < 100 ORDER BY name
{% endhighlight %}

Thread safety you don't really need to worry about, but chainability is core to how Sequel is generally used. Almost all dataset methods that affect the SQL produced return modified copies of the receiving dataset.

Another important thing to realise is that dataset methods that return modified datasets do not execute the dataset's code on the database. 

**Only dataset methods that return or yield results will execute the code on the database**:

{% highlight ruby %}
 # No SQL queries sent:
ds3 = ds.select(:id, :name).order(:name).filter{id < 100}

 # Until you call a method that returns results
results = ds3.all
{% endhighlight %}

One important consequence of this API style is that if you use a method chain that includes both methods that return modified copies and a method that executes the SQL, **the method that executes the SQL should generally be the last method in the chain**:

{% highlight ruby %}
 # Good
ds.select(:id, :name).order(:name).filter{id < 100}.all

 # Bad
ds.all.select(:id, :name).order(:name).filter{id < 100}

{% endhighlight %}


This is because all will return an array of hashes, and select, order, and filter are dataset methods, not array methods.


A dataset represents an SQL query, or more generally, an abstract set of rows in the database. Datasets can be used to create, retrieve, update and delete records.

Query results are always retrieved on demand, so a dataset can be kept around and reused indefinitely (datasets never cache results):

{% highlight ruby %}
my_posts = DB[:posts].filter(:author => 'david') #=> no records are retrieved
my_posts.all                                     #=> records are retrieved
my_posts.all                                     #=> records are retrieved again
{% endhighlight %}

Most dataset methods return modified copies of the dataset (functional style), so you can reuse different datasets to access data:

{% highlight ruby %}
posts = DB[:posts]
davids_posts = posts.filter(:author => 'david')
old_posts = posts.filter('stamp < ?', Date.today - 7)
davids_old_posts = davids_posts.filter('stamp < ?', Date.today - 7)
{% endhighlight %}

Datasets are Enumerable objects, so they can be manipulated using any of the Enumerable methods, such as map, inject, etc.


## Overview of Methods

Most Dataset methods that users will use can be broken down into two types:

* Methods that return modified datasets
* Methods that execute code on the database

### Methods that return modified datasets

Most dataset methods fall into this category, which can be further broken down by the clause they affect:

| SQL | Methods |
| --- | --- |
| SELECT | select, select_all, select_append, select_group, select_more |
| FROM | from, from_self |
| JOIN | join, left_join, right_join, full_join, natural_join, natural_left_join, natural_right_join, natural_full_join, cross_join, inner_join, left_outer_join, right_outer_join, full_outer_join, join_table |
| WHERE | where, filter, exclude, exclude_where, and, or, grep, invert, unfiltered |
| GROUP | group, group_by, group_and_count, select_group, ungrouped |
| HAVING | having, exclude_having, invert, unfiltered |
| ORDER | order, order_by, order_append, order_prepend, order_more, reverse, reverse_order, unordered |
| LIMIT/OFFSET | limit, offset, unlimited |
| compounds | union, intersect, except |
| locking | for_update, lock_style |
| common table expressions | with, with_recursive |
| other | clone, distinct, naked, qualify, server, with_sql |

---

## PUBLIC INSTANCE METHODS


### and (*cond, &block)

Alias for `where`.


<hr class="divider">

### clone (opts = nil)

Returns a new clone of the dataset with the given options merged. If the options changed include options in `COLUMN_CHANGE_OPTS`, the cached columns are deleted. 

**This method should generally not be called directly by user code.**

<hr class="divider">

### distinct (*args, &block)

Returns a copy of the dataset with the SQL `DISTINCT` clause. The `DISTINCT` clause is used to remove duplicate rows from the output. 

{% highlight ruby %}
DB[:items].distinct #=> SQL: SELECT DISTINCT * FROM items
{% endhighlight %}

If arguments are provided, uses a `DISTINCT ON` clause, in which case it will only be distinct on those columns, instead of all returned columns. 

{% highlight ruby %}
DB[:items].order(:id).distinct(:id) #=> SQL: SELECT DISTINCT ON (id) * FROM items ORDER BY id
{% endhighlight %}

If a block is given, it is treated as a virtual row block, similar to where. Raises an error if arguments are given and `DISTINCT ON` is not supported.

{% highlight ruby %}
DB[:items].order(:id).distinct { func(:id) } #=> SQL: SELECT DISTINCT ON (func(id)) * FROM items ORDER BY id
{% endhighlight %}


<hr class="divider">

### except (dataset, opts=OPTS)

Adds an `EXCEPT` clause using a second dataset object. An `EXCEPT` compound dataset returns all rows in the current dataset that are not in the given dataset. Raises an InvalidOperation if the operation is not supported. 

**Options:**

| Option | Description |
| --- | --- |
| :alias | Use the given value as the #from_self alias
| :all | Set to true to use EXCEPT ALL instead of EXCEPT, so duplicate rows can occur
| :from_self | Set to false to not wrap the returned dataset in a #from_self, use with care


{% highlight ruby %}
DB[:items].except(DB[:other_items])
 #=> SELECT * FROM (SELECT * FROM items EXCEPT SELECT * FROM other_items) AS t1

DB[:items].except(DB[:other_items], :all=>true, :from_self=>false)
 #=> SELECT * FROM items EXCEPT ALL SELECT * FROM other_items

DB[:items].except(DB[:other_items], :alias=>:i)
 #=> SELECT * FROM (SELECT * FROM items EXCEPT SELECT * FROM other_items) AS i
{% endhighlight %}


<hr class="divider">

### exclude (*cond, &block)

Performs the inverse of #where. Note that if you have multiple filter conditions, this is not the same as a negation of all conditions.

{% highlight ruby %}
DB[:items].exclude(:category => 'software')   #=> SELECT * FROM items WHERE (category != 'software')

DB[:items].exclude(:category => 'software', :id=>3)  #=> SELECT * FROM items WHERE ((category != 'software') OR (id != 3))
{% endhighlight %}


<hr class="divider">

### exclude_having (*cond, &block)

Inverts the given conditions and adds them to the HAVING clause.

{% highlight ruby %}
DB[:items].select_group(:name).exclude_having { count(name) < 2 }
 #=> SELECT name FROM items GROUP BY name HAVING (count(name) >= 2)
{% endhighlight %}

<hr class="divider">

### exclude_where (*cond, &block)

Alias for `exclude`.

<hr class="divider">

### extension (*exts)
Return a clone of the dataset loaded with the extensions, see `extension!`.

<hr class="divider">

### filter (*cond, &block)
Alias for where.

<hr class="divider">

### for_update ()
Returns a cloned dataset with a `:update` lock style.

{% highlight ruby %}
DB[:table].for_update  #=> SELECT * FROM table FOR UPDATE
{% endhighlight %}

<hr class="divider">

### from (*source, &block)

Returns a copy of the dataset with the source changed. 

{% highlight ruby %}
DB[:items].from(:blah) #=> SQL: SELECT * FROM blah
{% endhighlight %}

If no source is given, removes all tables. 

{% highlight ruby %}
DB[:items].from    #=> SQL: SELECT *
{% endhighlight %}

If multiple sources are given, it is the same as using a `CROSS JOIN` (cartesian product) between all tables. 

{% highlight ruby %}
DB[:items].from(:blah, :foo) #=> SQL: SELECT * FROM blah, foo
{% endhighlight %}

If a block is given, it is treated as a virtual row block, similar to where.

{% highlight ruby %}
DB[:items].from { fun(arg) } #=> SQL: SELECT * FROM fun(arg)
{% endhighlight %}

<hr class="divider">

### from_self (opts=OPTS)

Returns a dataset selecting from the current dataset. 

{% highlight ruby %}
ds = DB[:items].order(:name).select(:id, :name)
 #=> SELECT id,name FROM items ORDER BY name

ds.from_self  #=> SELECT * FROM (SELECT id, name FROM items ORDER BY name) AS t1
{% endhighlight %}


Supplying the `:alias` option controls the alias of the result.

{% highlight ruby %}
ds.from_self(:alias=>:foo)
 #=> SELECT * FROM (SELECT id, name FROM items ORDER BY name) AS foo

ds.from_self(:alias=>:foo, :column_aliases=>[:c1, :c2])
 #=> SELECT * FROM (SELECT id, name FROM items ORDER BY name) AS foo(c1, c2)
{% endhighlight %}

<hr class="divider">

### grep (columns, patterns, opts=OPTS)

Match any of the columns to any of the patterns. 

The terms can be strings (which use LIKE) or regular expressions (which are only supported on MySQL and PostgreSQL). Note that the total number of pattern matches will be Array(columns).length * Array(terms).length, which could cause performance issues.

**Options (all are boolean):**

| Option | Description |
| --- | --- |
| :all_columns | All columns must be matched to any of the given patterns.
| :all_patterns | All patterns must match at least one of the columns.
| :case_insensitive | Use a case insensitive pattern match (the default is case sensitive if the database supports it).

If both `:all_columns` and `:all_patterns` are **true**, all columns must match all patterns.

**Examples:**

{% highlight ruby %}
dataset.grep(:a, '%test%')    #=> SELECT * FROM items WHERE (a LIKE '%test%' ESCAPE '\')

dataset.grep([:a, :b], %w'%test% foo')  
 #=> SELECT * FROM items WHERE ((a LIKE '%test%' ESCAPE '\') OR (a LIKE 'foo' ESCAPE '\')
 #=>   OR (b LIKE '%test%' ESCAPE '\') OR (b LIKE 'foo' ESCAPE '\'))

dataset.grep([:a, :b], %w'%foo% %bar%', :all_patterns=>true)
 #=> SELECT * FROM a WHERE (((a LIKE '%foo%' ESCAPE '\') OR (b LIKE '%foo%' ESCAPE '\'))
 #=>   AND ((a LIKE '%bar%' ESCAPE '\') OR (b LIKE '%bar%' ESCAPE '\')))

dataset.grep([:a, :b], %w'%foo% %bar%', :all_columns=>true)
 #=> SELECT * FROM a WHERE (((a LIKE '%foo%' ESCAPE '\') OR (a LIKE '%bar%' ESCAPE '\'))
 #=>   AND ((b LIKE '%foo%' ESCAPE '\') OR (b LIKE '%bar%' ESCAPE '\')))

dataset.grep([:a, :b], %w'%foo% %bar%', :all_patterns=>true, :all_columns=>true)
  #=> SELECT * FROM a WHERE ((a LIKE '%foo%' ESCAPE '\') AND (b LIKE '%foo%' ESCAPE '\')
  #=>   AND (a LIKE '%bar%' ESCAPE '\') AND (b LIKE '%bar%' ESCAPE '\'))
{% endhighlight %}


<hr class="divider">


### group (*columns, &block)

Returns a copy of the dataset with the results grouped by the value of the given columns. 

{% highlight ruby %}
DB[:items].group(:id)         #=> SELECT * FROM items GROUP BY id
DB[:items].group(:id, :name)  #=> SELECT * FROM items GROUP BY id, name
{% endhighlight %}


If a block is given, it is treated as a virtual row block, similar to where.

{% highlight ruby %}
DB[:items].group { [a, sum(b)] }   #=> SELECT * FROM items GROUP BY a, sum(b)
{% endhighlight %}


<hr class="divider">

### group_and_count (*columns, &block)

Returns a dataset grouped by the given column with count by group. 

{% highlight ruby %}
DB[:items].group_and_count(:name).all   #=> SELECT name, count(*) AS count FROM items GROUP BY name 
   # => [{:name=>'a', :count=>1}, ...]

DB[:items].group_and_count(:first_name, :last_name).all
 #=> SELECT first_name, last_name, count(*) AS count FROM items GROUP BY first_name, last_name
  # => [{:first_name=>'a', :last_name=>'b', :count=>1}, ...]
{% endhighlight %}

Column aliases may be supplied, and will be included in the select clause. 

{% highlight ruby %}
DB[:items].group_and_count(:first_name___name).all  #=> SELECT first_name AS name, count(*) AS count FROM items GROUP BY first_name
  # => [{:name=>'a', :count=>1}, ...]
{% endhighlight %}


If a block is given, it is treated as a virtual row block, similar to where.

{% highlight ruby %}
DB[:items].group_and_count { substr(first_name, 1, 1).as(initial) }.all   #=> SELECT substr(first_name, 1, 1) AS initial, count(*) AS count FROM items GROUP BY substr(first_name, 1, 1)
  # => [{:initial=>'a', :count=>1}, ...]
{% endhighlight %}


<hr class="divider">

### group_by (*columns, &block)

Alias of group

<hr class="divider">

### group_cube ()

Adds the appropriate `CUBE` syntax to `GROUP BY`.

<hr class="divider">

### group_rollup ()

Adds the appropriate `ROLLUP` syntax to `GROUP BY`.

<hr class="divider">

### having (*cond, &block)

Returns a copy of the dataset with the `HAVING` conditions changed. See where for argument types.

{% highlight ruby %}
DB[:items].group(:sum).having(:sum=>10)   #=> SELECT * FROM items GROUP BY sum HAVING (sum = 10)
{% endhighlight %}

<hr class="divider">

### intersect (dataset, opts=OPTS)

Adds an `INTERSECT` clause using a second dataset object. An `INTERSECT` compound dataset returns all rows in both the current dataset and the given dataset. 

Raises an `InvalidOperation` if the operation is not supported. 

**Options:**

| Option | Description |
| --- | --- |
| :alias	| Use the given value as the #from_self alias
| :all	| Set to true to use INTERSECT ALL instead of INTERSECT, so duplicate rows can occur
| :from_self	| Set to false to not wrap the returned dataset in a #from_self, use with care.


{% highlight ruby %}
DB[:items].intersect(DB[:other_items])
 #=> SELECT * FROM (SELECT * FROM items INTERSECT SELECT * FROM other_items) AS t1

DB[:items].intersect(DB[:other_items], :all => true, :from_self => false)
 #=> SELECT * FROM items INTERSECT ALL SELECT * FROM other_items

DB[:items].intersect(DB[:other_items], :alias => :i)
 #=> SELECT * FROM (SELECT * FROM items INTERSECT SELECT * FROM other_items) AS i
{% endhighlight %}


<hr class="divider">

### invert ()

Inverts the current `WHERE` and `HAVING` clauses. 

If there is neither a `WHERE` or `HAVING` clause, adds a `WHERE` clause that is always false.

{% highlight ruby %}
DB[:items].where(:category => 'software').invert
 #=> SELECT * FROM items WHERE (category != 'software')

DB[:items].where(:category => 'software', :id => 3).invert
 #=> SELECT * FROM items WHERE ((category != 'software') OR (id != 3))
{% endhighlight %}


<hr class="divider">

### join (*args, &block)

Alias of inner_join

<hr class="divider">

### join_table (type, table, expr=nil, options=OPTS, &block)

Returns a joined dataset. Not usually called directly, users should use the appropriate join method (e.g. join, left_join, natural_join, cross_join) which fills in the type argument.

Takes the following arguments:

| Option | Description |
| ---  | --- |
|type  | The type of join to do (e.g. :inner)
|table | table to join into the current dataset. Generally one of the **Table types** (below):
|expr | conditions used when joining, depends on type. See **Expression Condition types** (below):
|options | a hash of options, with the following Supported Option Keys (below):
|block | The block argument should only be given if a JOIN with an ON clause is used, in which case it yields the table alias/name for the table currently being joined, the table alias/name for the last joined (or first table), and an array of previous SQL::JoinClause. Unlike where, this block is not treated as a virtual row block.

| Table types ||
| --- | --- |
| String, Symbol | identifier used as table or view name
| Dataset | a subselect is performed with an alias of tN for some value of N
| SQL::Function	| set returning function
| SQL::AliasedExpression | already aliased expression. Uses given alias unless overridden by the `:table_alias` option. 


| Expression Condition types ||
| --- | --- |
| Hash, Array of pairs | Assumes key (1st arg) is column of joined table (unless already qualified), and value (2nd arg) is column of the last joined or primary table (or the :implicit_qualifier option). To specify multiple conditions on a single joined table column, you must use an array. Uses a JOIN with an ON clause.
| Array | If all members of the array are symbols, considers them as columns and uses a JOIN with a USING clause. Most databases will remove duplicate columns from the result set if this is used.
| nil | If a block is not given, doesn't use ON or USING, so the JOIN should be a NATURAL or CROSS join. If a block is given, uses an ON clause based on the block, see below.
| otherwise | Treats the argument as a filter expression, so strings are considered literal, symbols specify boolean columns, and Sequel expressions can be used. Uses a JOIN with an ON clause.


| Supported Option Keys ||
| --- | --- |
| :table_alias | Override the table alias used when joining. In general you shouldn't use this option, you should provide the appropriate SQL::AliasedExpression as the table argument.
| :implicit_qualifier | The name to use for qualifying implicit conditions. By default, the last joined or primary table is used.
| :reset_implicit_qualifier | Can set to false to ignore this join when future joins determine qualifier for implicit conditions.
| :qualify | Can be set to false to not do any implicit qualification. Can be set to :deep to use the Qualifier AST Transformer, which will attempt to qualify subexpressions of the expression tree. Can be set to :symbol to only qualify symbols. Defaults to the value of default_join_table_qualification.

**Examples:**

{% highlight ruby %}
DB[:a].join_table(:cross, :b)             #=> SELECT * FROM a CROSS JOIN b

DB[:a].join_table(:inner, DB[:b], :c=>d)  #=> SELECT * FROM a INNER JOIN (SELECT * FROM b) AS t1 ON (t1.c = a.d)

DB[:a].join_table(:left, :b___c, [:d])    #=> SELECT * FROM a LEFT JOIN b AS c USING (d)

DB[:a].natural_join(:b).join_table(:inner, :c) do |ta, jta, js|
  (Sequel.qualify(ta, :d) > Sequel.qualify(jta, :e)) & {Sequel.qualify(ta, :f)=>DB.from(js.first.table).select(:g)}
end
 #=> SELECT * FROM a NATURAL JOIN b INNER JOIN c ON ((c.d > b.e) AND (c.f IN (SELECT g FROM b)))
{% endhighlight %}


<hr class="divider">

### lateral ()

Marks this dataset as a lateral dataset. If used in another dataset's FROM or JOIN clauses, it will surround the subquery with LATERAL to enable it to deal with previous tables in the query:

{% highlight ruby %}
DB.from(:a, DB[:b].where(:a__c=>:b__d).lateral)   #=> SELECT * FROM a, LATERAL (SELECT * FROM b WHERE (a.c = b.d))
{% endhighlight %}


<hr class="divider">

### limit (l, o = (no_offset = true; nil))

If given an integer, the dataset will contain only the first n results. 

{% highlight ruby %}
DB[:items].limit(10)          #=> SELECT * FROM items LIMIT 10 
{% endhighlight %}

If a second argument is given, it is used as an offset. 

{% highlight ruby %}
DB[:items].limit(10, 20)      #=> SELECT * FROM items LIMIT 10 OFFSET 20
{% endhighlight %}

If given a range, it will contain only those at offsets within that range. 

{% highlight ruby %}
 # NOTE! three (3) dots 
DB[:items].limit(10...20)     #=> SELECT * FROM items LIMIT 10 OFFSET 10

 # NOTE! two (2) dots only
DB[:items].limit(10..20)      #=> SELECT * FROM items LIMIT 11 OFFSET 10
{% endhighlight %}


To use an offset without a limit, pass nil as the first argument.

{% highlight ruby %}
DB[:items].limit(nil, 20)     #=> SELECT * FROM items OFFSET 20
{% endhighlight %}


<hr class="divider">

### lock_style (style)

Returns a cloned dataset with the given lock style. If style is a string, it will be used directly. 

{% highlight ruby %}
DB[:items].lock_style('FOR SHARE NOWAIT')  #=> SELECT * FROM items FOR SHARE NOWAIT
{% endhighlight %}

**NOTE!!! You should never pass a string to this method that is derived from user input, as that can lead to SQL injection.**

A symbol may be used for database independent locking behaviour, but all supported symbols have separate methods (e.g. #for_update).

<hr class="divider">

### naked ()

Returns a cloned dataset without a row_proc.

{% highlight ruby %}
ds = DB[:items]
ds.row_proc = proc { |r| r.invert }
ds.all # => [{2=>:id}]
ds.naked.all # => [ {:id=>2} ]
{% endhighlight %}


<hr class="divider">

### offset (o)

Returns a copy of the dataset with a specified order. 

{% highlight ruby %}
DB[:items].offset(10)  #=> SELECT * FROM items OFFSET 10
{% endhighlight %}

Can be safely combined with limit, however the `limit()` offset overrides the `offset()` if you've called `offset()` first.

{% highlight ruby %}
DB[:items].offset(10).limit(10,20)  #=> SELECT * FROM items LIMIT 10 OFFSET 20

 # NOTE! order of limit().offset()
DB[:items].limit(10,20).offset(10)  #=> SELECT * FROM items LIMIT 10 OFFSET 10
{% endhighlight %}


<hr class="divider">

### or (*cond, &block)

Adds an alternate filter to an existing filter using OR. If no filter exists an Error is raised.

{% highlight ruby %}
DB[:items].where(:a).or(:b)  #=> SELECT * FROM items WHERE a OR b
{% endhighlight %}

<hr class="divider">

### order (*columns, &block)

Returns a copy of the dataset with the order changed. 

{% highlight ruby %}
DB[:items].order(:name)     #=> SELECT * FROM items ORDER BY name
DB[:items].order(:a, :b)    #=> SELECT * FROM items ORDER BY a, b
{% endhighlight %}


If the dataset has an existing order, it is ignored and overwritten with this order. 

{% highlight ruby %}
DB[:items].order(Sequel.lit('a + b'))   #=> SELECT * FROM items ORDER BY a + b
DB[:items].order(:a + :b)               #=> SELECT * FROM items ORDER BY (a + b)
{% endhighlight %}


If a nil is given the returned dataset has no order. 

{% highlight ruby %}
DB[:items].order(nil)   #=> SELECT * FROM items
{% endhighlight %}


This can accept multiple arguments of varying kinds, such as SQL functions. 

{% highlight ruby %}
DB[:items].order( Sequel.desc(:name) )                #=> SELECT * FROM items ORDER BY name DESC
DB[:items].order( Sequel.asc(:name, :nulls=>:last) )  #=> SELECT * FROM items ORDER BY name ASC NULLS LAST
{% endhighlight %}


If a block is given, it is treated as a virtual row block, similar to where.

{% highlight ruby %}
DB[:items].order{sum(name).desc}    #=> SELECT * FROM items ORDER BY sum(name) DESC
{% endhighlight %}


<hr class="divider">

### order_append (*columns, &block)

Alias of `#order_more`, for naming consistency with `order_prepend`.

<hr class="divider">

###  order_by (*columns, &block)
Alias of `order`

<hr class="divider">

### order_more (*columns, &block)

Returns a copy of the dataset with the order columns added to the end of the existing order.

{% highlight ruby %}
DB[:items].order(:a).order(:b)        #=> SELECT * FROM items ORDER BY b
DB[:items].order(:a).order_more(:b)   #=> SELECT * FROM items ORDER BY a, b
{% endhighlight %}

<hr class="divider">

###  order_prepend (*columns, &block)

Returns a copy of the dataset with the order columns added to the beginning of the existing order.

{% highlight ruby %}
DB[:items].order(:a).order(:b)            #=> SELECT * FROM items ORDER BY b
DB[:items].order(:a).order_prepend(:b)    #=> SELECT * FROM items ORDER BY b, a
{% endhighlight %}


<hr class="divider">

### qualify (table=first_source)

Qualify to the given table, or first source if no table is given.

{% highlight ruby %}
DB[:items].where(:id=>1).qualify        #=> SELECT items.* FROM items WHERE (items.id = 1)

DB[:items].where(:id=>1).qualify(:i)    #=> SELECT i.* FROM items WHERE (i.id = 1)
{% endhighlight %}
 
 
<hr class="divider">

### returning (*values)

Modify the RETURNING clause, only supported on a few databases. If returning is used, instead of insert returning the autogenerated primary key or update/delete returning the number of modified rows, results are returned using fetch_rows.

{% highlight ruby %}
DB[:items].returning              #=> RETURNING *
DB[:items].returning(nil)         #=> RETURNING NULL
DB[:items].returning(:id, :name)  #=> RETURNING id, name
{% endhighlight %}

<hr class="divider">

### reverse (*order, &block)

Returns a copy of the dataset with the order reversed. If no order is given, the existing order is inverted.

{% highlight ruby %}
DB[:items].reverse(:id)                             #=> SELECT * FROM items ORDER BY id DESC
DB[:items].reverse{foo(bar)}                        #=> SELECT * FROM items ORDER BY foo(bar) DESC
DB[:items].order(:id).reverse                       #=> SELECT * FROM items ORDER BY id DESC
DB[:items].order(:id).reverse(Sequel.desc(:name))   #=> SELECT * FROM items ORDER BY name ASC
{% endhighlight %}

<hr class="divider">

###  reverse_order (*order, &block)
Alias of `reverse`

<hr class="divider">

### select (*columns, &block)

Returns a copy of the dataset with the columns selected changed to the given columns. This also takes a virtual row block, similar to `where`.

{% highlight ruby %}
DB[:items].select(:a)                #=> SELECT a FROM items
DB[:items].select(:a, :b)            #=> SELECT a, b FROM items
DB[:items].select { [a, sum(b)] }    #=> SELECT a, sum(b) FROM items
{% endhighlight %}


<hr class="divider">

### select_all (*tables)

Returns a copy of the dataset selecting the wildcard if no arguments are given. If arguments are given, treat them as tables and select all columns (using the wildcard) from each table.

{% highlight ruby %}
DB[:items].select(:a).select_all        #=> SELECT * FROM items
DB[:items].select_all(:items)           #=> SELECT items.* FROM items
DB[:items].select_all(:items, :foo)     #=> SELECT items.*, foo.* FROM items
{% endhighlight %}


<hr class="divider">

### select_append (*columns, &block)

Returns a copy of the dataset with the given columns added to the existing selected columns. If no columns are currently selected, it will select the columns given in addition to *.

{% highlight ruby %}
DB[:items].select(:a).select(:b)            #=> SELECT b FROM items
DB[:items].select(:a).select_append(:b)     #=> SELECT a, b FROM items
DB[:items].select_append(:b)                #=> SELECT *, b FROM items
{% endhighlight %}

<hr class="divider">

### select_group (*columns, &block)

Set both the select and group clauses with the given columns. Column aliases may be supplied, and will be included in the select clause. This also takes a virtual row block similar to where.

{% highlight ruby %}
DB[:items].select_group(:a, :b)            #=> SELECT a, b FROM items GROUP BY a, b

DB[:items].select_group(:c___a) { f(c2) }  #=> SELECT c AS a, f(c2) FROM items GROUP BY c, f(c2)
{% endhighlight %}

<hr class="divider">

###  select_more (*columns, &block)
Alias for `select_append`.

<hr class="divider">

### server (servr)

Set the server for this dataset to use. Used to pick a specific database shard to run a query against, or to override the default (where SELECT uses :read_only database and all other queries use the :default database). 

This method is always available but is only useful when database sharding is being used.

{% highlight ruby %}
DB[:items].all                      #=> Uses the :read_only or :default server 
DB[:items].delete                   #=> Uses the :default server
DB[:items].server(:blah).delete     #=> Uses the :blah server
{% endhighlight %}


<hr class="divider">

### server? (server)

If the database uses sharding and the current dataset has not had a server set, return a cloned dataset that uses the given server. Otherwise, return the receiver directly instead of returning a clone.

<hr class="divider">

### unbind ()

Unbind bound variables from this dataset's filter and return an array of two objects. The first object is a modified dataset where the filter has been replaced with one that uses bound variable placeholders. The second object is the hash of unbound variables. You can then prepare and execute (or just call) the dataset with the bound variables to get results.

{% highlight ruby %}
ds, bv = DB[:items].where(:a=>1).unbind
ds  #=> SELECT * FROM items WHERE (a = $a)
bv  #=>  {:a => 1}
ds.call(:select, bv)
{% endhighlight %}


<hr class="divider">

### unfiltered ()

Returns a copy of the dataset with no filters (HAVING or WHERE clause) applied.

{% highlight ruby %}
DB[:items].group(:a).having(:a=>1).where(:b).unfiltered  #=> SELECT * FROM items GROUP BY a
{% endhighlight %}


<hr class="divider">

### ungrouped ()
Returns a copy of the dataset with no grouping (GROUP or HAVING clause) applied.

{% highlight ruby %}
DB[:items].group(:a).having(:a=>1).where(:b).ungrouped
 #=> SELECT * FROM items WHERE b
{% endhighlight %}
 
<hr class="divider">

### union (dataset, opts=OPTS)

Adds a UNION clause using a second dataset object. A UNION compound dataset returns all rows in either the current dataset or the given dataset. 

**Options:**

| Option | Description |
| --- | --- |
| :alias | Use the given value as the #from_self alias
| :all | Set to true to use UNION ALL instead of UNION, so duplicate rows can occur
| :from_self | Set to false to not wrap the returned dataset in a #from_self, use with care.


{% highlight ruby %}
DB[:items].union(DB[:other_items])
 #=> SELECT * FROM (SELECT * FROM items UNION SELECT * FROM other_items) AS t1

DB[:items].union(DB[:other_items], :all=>true, :from_self=>false)
 #=> SELECT * FROM items UNION ALL SELECT * FROM other_items

DB[:items].union(DB[:other_items], :alias=>:i)
 #=> SELECT * FROM (SELECT * FROM items UNION SELECT * FROM other_items) AS i
{% endhighlight %}


<hr class="divider">

### unlimited ()
Returns a copy of the dataset with no limit or offset.

{% highlight ruby %}
DB[:items].limit(10, 20).unlimited     #=> SELECT * FROM items
{% endhighlight %}


<hr class="divider">

### unordered ()
Returns a copy of the dataset with no order.

{% highlight ruby %}
DB[:items].order(:a).unordered # SELECT * FROM items
{% endhighlight %}

<hr class="divider">

### where (*cond, &block)

Returns a copy of the dataset with the given `WHERE` conditions imposed upon it.

Accepts the following argument types:


| Description ||
| --- | --- |
| **Hash** | list of equality/inclusion expressions
| **Array** | depends: <hr class="divider">
<ul><li>If first member is a string, assumes the rest of the arguments are parameters and interpolates them into the string.</li><li>If all members are arrays of length two, treats the same way as a hash, except it allows for duplicate keys to be specified.</li><li>Otherwise, treats each argument as a separate condition.</li></ul>
| **String** | taken literally
| **Symbol** | taken as a boolean column argument (e.g. WHERE active)
| **Sequel::SQL::BooleanExpression** | an existing condition expression, probably created using the Sequel expression filter DSL.

`where` also accepts a block, which should return one of the above argument types, and is treated the same way. This block yields a virtual row object, which is easy to use to create identifiers and functions. For more details on the virtual row support, see the “Virtual Rows” guide

If both a block and regular argument are provided, they get ANDed together.

**Examples:**

{% highlight ruby %}
DB[:items].where(:id => 3)          #=> SELECT * FROM items WHERE (id = 3)

DB[:items].where('price < ?', 100)  #=> SELECT * FROM items WHERE price < 100

DB[:items].where([[:id, [1,2,3]], [:id, 0..10]])
 #=> SELECT * FROM items WHERE ((id IN (1, 2, 3)) AND ((id >= 0) AND (id <= 10)))

DB[:items].where('price < 100')     #=> SELECT * FROM items WHERE price < 100

DB[:items].where(:active)           #=> SELECT * FROM items WHERE :active

DB[:items].where { price < 100 }    #=> SELECT * FROM items WHERE (price < 100)
{% endhighlight %}


Multiple where calls can be chained for scoping:

{% highlight ruby %}
software = dataset.where(:category => 'software').where { price < 100 }
 #=> SELECT * FROM items WHERE ((category = 'software') AND (price < 100))
{% endhighlight %}


See the “Dataset Filtering” guide for more examples and details.

<hr class="divider">

### with (name, dataset, opts=OPTS)

Add a common table expression (CTE) with the given name and a dataset that defines the CTE. A common table expression acts as an inline view for the query. 


**Options:**

| Option | Description |
| --- | --- |
| :args | Specify the arguments/columns for the CTE, should be an array of symbols.
| :recursive | Specify that this is a recursive CTE


{% highlight ruby %}
DB[:items].with(:items, DB[:syx].where(:name.like('A%')))
  #=> WITH items AS (SELECT * FROM syx WHERE (name LIKE 'A%' ESCAPE '\')) SELECT * FROM items
{% endhighlight %}


<hr class="divider">

### with_recursive (name, nonrecursive, recursive, opts=OPTS)

Add a recursive common table expression (CTE) with the given name, a dataset that defines the nonrecursive part of the CTE, and a dataset that defines the recursive part of the CTE. 

**Options:**

| Option | Description |
| --- | --- |
| :args	| Specify the arguments/columns for the CTE, should be an array of symbols.
| :union_all	| Set to false to use UNION instead of UNION ALL combining the nonrecursive and recursive parts.


{% highlight ruby %}
DB[:t].with_recursive(:t,
  DB[:i1].select(:id, :parent_id).where(:parent_id=>nil),
  DB[:i1].join(:t, :id=>:parent_id).select(:i1__id, :i1__parent_id),
  :args=>[:id, :parent_id])

  # WITH RECURSIVE "t"("id", "parent_id") AS (
  #   SELECT "id", "parent_id" FROM "i1" WHERE ("parent_id" IS NULL)
  #   UNION ALL
  #   SELECT "i1"."id", "i1"."parent_id" FROM "i1" INNER JOIN "t" ON ("t"."id" = "i1"."parent_id")
  # ) SELECT * FROM "t"

{% endhighlight %}


<hr class="divider">

###  with_sql (sql, *args)

Returns a copy of the dataset with the static SQL used. This is useful if you want to keep the same row_proc/graph, but change the SQL used to custom SQL.

{% highlight ruby %}
DB[:items].with_sql('SELECT * FROM foo') #=> SELECT * FROM foo
{% endhighlight %}

You can use placeholders in your SQL and provide arguments for those placeholders:

{% highlight ruby %}
DB[:items].with_sql('SELECT ? FROM foo', 1) #=> SELECT 1 FROM foo
{% endhighlight %}

You can also provide a method name and arguments to call to get the SQL:

{% highlight ruby %}
DB[:items].with_sql(:insert_sql, :b=>1) #=> INSERT INTO items (b) VALUES (1)
{% endhighlight %}


---

## Protected Instance methods

### compound_clone (type, dataset, opts)
Add the dataset to the list of compounds

<hr class="divider">

### options_overlap (opts)
Return true if the dataset has a non-nil value for any key in opts.

<br>
### simple_select_all? ()

Whether this dataset is a simple select from an underlying table, such as:

{% highlight ruby %}
SELECT * FROM table
SELECT table.* FROM table
{% endhighlight %}


--- 

### Methods that execute code on the database

Most other dataset methods commonly used will execute the dataset's SQL on the database:

| SQL | Methods |
| --- | --- |
| SELECT (All Records) | all, each, map, to_hash, to_hash_groups, select_map, select_order_map, select_hash, select_hash_groups
| SELECT (First Record)	| first, last, [], single_record
| SELECT (Single Value)	| get, single_value
| SELECT (Aggregates)	| count, avg, max, min, sum, range, interval
| INSERT	| insert, <<, import, multi_insert
| UPDATE	| update
| DELETE	| delete
| other	| columns, columns!, truncate

---

## Public Instance methods

### << (arg)

Inserts the given argument into the database. Returns self so it can be used safely when chaining:

{% highlight ruby %}
DB[:items] << {:id=>0, :name=>'Zero'} << DB[:old_items].select(:id, name)
{% endhighlight %}


<hr class="divider">
### [] (*conditions)

Returns the first record matching the conditions. Examples:

{% highlight ruby %}
DB[:table][:id=>1] # SELECT * FROM table WHERE (id = 1) LIMIT 1
 # => {:id=1}
{% endhighlight %}

<hr class="divider">
### all (&block)

Returns an array with all records in the dataset. If a block is given, the array is iterated over after all items have been loaded.

{% highlight ruby %}
DB[:table].all # SELECT * FROM table
 # => [{:id=>1, ...}, {:id=>2, ...}, ...]

 # Iterate over all rows in the table
DB[:table].all{|row| p row}
{% endhighlight %}

<hr class="divider">
###  avg (column=Sequel.virtual_row(&Proc.new))
Returns the average value for the given column/expression. Uses a virtual row block if no argument is given.

{% highlight ruby %}
DB[:table].avg(:number)           #=> SELECT avg(number) FROM table LIMIT 1
 # => 3
DB[:table].avg{function(column)}  #=> SELECT avg(function(column)) FROM table LIMIT 1
 # => 1
{% endhighlight %}

<hr class="divider">
### columns ()

Returns the columns in the result set in order as an array of symbols. If the columns are currently cached, returns the cached value. Otherwise, a SELECT query is performed to retrieve a single row in order to get the columns.

If you are looking for all columns for a single table and maybe some information about each column (e.g. database type), see Database#schema.

{% highlight ruby %}
DB[:table].columns   # => [:id, :name]
{% endhighlight %}


<hr class="divider">
### columns! ()

Ignore any cached column information and perform a query to retrieve a row in order to get the columns.

{% highlight ruby %}
DB[:table].columns!  # => [:id, :name]
{% endhighlight %}

<hr class="divider">

### count (arg=(no_arg=true), &block)

Returns the number of records in the dataset. If an argument is provided, it is used as the argument to count. If a block is provided, it is treated as a virtual row, and the result is used as the argument to count.

{% highlight ruby %}
DB[:table].count                    #=> SELECT count(*) AS count FROM table LIMIT 1
 # => 3
DB[:table].count(:column)           #=> SELECT count(column) AS count FROM table LIMIT 1
 # => 2
DB[:table].count{foo(column)}       #=> SELECT count(foo(column)) AS count FROM table LIMIT 1
 # => 1
{% endhighlight %}

<hr class="divider">
### delete (&block)

Deletes the records in the dataset. The returned value should be number of records deleted, but that is adapter dependent.

{% highlight ruby %}
DB[:table].delete     #=> DELETE * FROM table
  # => 3
{% endhighlight %}


<hr class="divider">
### each ()
Iterates over the records in the dataset as they are yielded from the database adapter, and returns self.

{% highlight ruby %}
DB[:table].each{|row| p row}      #=> SELECT * FROM table
{% endhighlight %}


Note that this method is not safe to use on many adapters if you are running additional queries inside the provided block. If you are running queries inside the block, you should use all instead of each for the outer queries, or use a separate thread or shard inside each.


<hr class="divider">
### empty? ()

Returns true if no records exist in the dataset, false otherwise

{% highlight ruby %}
DB[:table].empty?       #=> SELECT 1 AS one FROM table LIMIT 1
 # => false
{% endhighlight %}

<hr class="divider">
### first (*args, &block)

If a integer argument is given, it is interpreted as a limit, and then returns all matching records up to that limit. If no argument is passed, it returns the first matching record. If any other type of argument(s) is passed, it is given to filter and the first matching record is returned. If a block is given, it is used to filter the dataset before returning anything.

If there are no records in the dataset, returns nil (or an empty array if an integer argument is given).

**Examples:**

{% highlight ruby %}
DB[:table].first                        #=> SELECT * FROM table LIMIT 1
 # => {:id=>7}

DB[:table].first(2)                     #=> SELECT * FROM table LIMIT 2
 # => [{:id=>6}, {:id=>4}]

DB[:table].first(:id=>2)                #=> SELECT * FROM table WHERE (id = 2) LIMIT 1
 # => {:id=>2}

DB[:table].first("id = 3")              #=> SELECT * FROM table WHERE (id = 3) LIMIT 1
 # => {:id=>3}

DB[:table].first("id = ?", 4)           #=> SELECT * FROM table WHERE (id = 4) LIMIT 1
 # => {:id=>4}

DB[:table].first{id > 2}                #=> SELECT * FROM table WHERE (id > 2) LIMIT 1
 # => {:id=>5}

DB[:table].first("id > ?", 4){id < 6}   #=> SELECT * FROM table WHERE ((id > 4) AND (id < 6)) LIMIT 1
 # => {:id=>5}

DB[:table].first(2){id < 2}             #=> SELECT * FROM table WHERE (id < 2) LIMIT 2
 # => [{:id=>1}]

{% endhighlight %}

<hr class="divider">

### first! (*args, &block)

Calls first. If first returns nil (signaling that no row matches), raise a Sequel::NoMatchingRow exception.

<hr class="divider">
### get (column=(no_arg=true; nil), &block)

Return the column value for the first matching record in the dataset. Raises an error if both an argument and block is given.

{% highlight ruby %}
DB[:table].get(:id)         #=> SELECT id FROM table LIMIT 1
 # => 3

ds.get{sum(id)}             #=> SELECT sum(id) AS v FROM table LIMIT 1
 # => 6
{% endhighlight %}

You can pass an array of arguments to return multiple arguments, but you must make sure each element in the array has an alias that Sequel can determine:

{% highlight ruby %}
DB[:table].get([:id, :name])              #=> SELECT id, name FROM table LIMIT 1
 # => [3, 'foo']
DB[:table].get{[sum(id).as(sum), name]}   #=> SELECT sum(id) AS sum, name FROM table LIMIT 1
 # => [6, 'foo']
{% endhighlight %}


<hr class="divider">
### import (columns, values, opts=OPTS)

Inserts multiple records into the associated table. This method can be used to efficiently insert a large number of records into a table in a single query if the database supports it. Inserts are automatically wrapped in a transaction.

This method is called with a columns array and an array of value arrays:

{% highlight ruby %}
DB[:table].import([:x, :y], [[1, 2], [3, 4]])
 # INSERT INTO table (x, y) VALUES (1, 2) 
 # INSERT INTO table (x, y) VALUES (3, 4)
{% endhighlight %}

This method also accepts a dataset instead of an array of value arrays:

{% highlight ruby %}
DB[:table].import([:x, :y], DB[:table2].select(:a, :b))     #=> INSERT INTO table (x, y) SELECT a, b FROM table2
{% endhighlight %}

| **Options:** ||
| --- | --- |
| :commit_every | Open a new transaction for every given number of records. For example, if you provide a value of 50, will commit after every 50 records.
| :return	| When the :value is :primary_key, returns an array of autoincremented primary key values for the rows inserted.
| :server	| Set the server/shard to use for the transaction and insert queries.
| :slice	| Same as :commit_every, :commit_every takes precedence.

<hr class="divider">
### insert (*values, &block)

Inserts values into the associated table. The returned value is generally the value of the primary key for the inserted row, but that is adapter dependent.

| insert handles a number of different argument formats:| |
| --- | --- |
| no arguments or single empty hash	| Uses DEFAULT VALUES
| single hash	| Most common format, treats keys as columns an values as values
| single array	| Treats entries as values, with no columns
| two arrays	| Treats first array as columns, second array as values
| single Dataset	| Treats as an insert based on a selection from the dataset given, with no columns
| array and dataset	| Treats as an insert based on a selection from the dataset given, with the columns given by the array.


**Examples:**

{% highlight ruby %}
DB[:items].insert                             #=> INSERT INTO items DEFAULT VALUES

DB[:items].insert({})                         #=> INSERT INTO items DEFAULT VALUES

DB[:items].insert([1,2,3])                    #=> INSERT INTO items VALUES (1, 2, 3)

DB[:items].insert([:a, :b], [1,2])            #=> INSERT INTO items (a, b) VALUES (1, 2)

DB[:items].insert(:a => 1, :b => 2)           #=> INSERT INTO items (a, b) VALUES (1, 2)

DB[:items].insert(DB[:old_items])             #=> INSERT INTO items SELECT * FROM old_items

DB[:items].insert([:a, :b], DB[:old_items])   #=> INSERT INTO items (a, b) SELECT * FROM old_items
{% endhighlight %}


<hr class="divider">
### interval (column=Sequel.virtual_row(&Proc.new))

Returns the interval between minimum and maximum values for the given column/expression. Uses a virtual row block if no argument is given.

{% highlight ruby %}
DB[:table].interval(:id) # SELECT (max(id) - min(id)) FROM table LIMIT 1
 # => 6
DB[:table].interval{function(column)} # SELECT (max(function(column)) - min(function(column))) FROM table LIMIT 1
 # => 7
{% endhighlight %}

<hr class="divider">
### last (*args, &block)

Reverses the order and then runs first with the given arguments and block. Note that this will not necessarily give you the last record in the dataset, unless you have an unambiguous order. If there is not currently an order for this dataset, raises an Error.

{% highlight ruby %}
DB[:table].order(:id).last # SELECT * FROM table ORDER BY id DESC LIMIT 1
 # => {:id=>10}

DB[:table].order(Sequel.desc(:id)).last(2) # SELECT * FROM table ORDER BY id ASC LIMIT 2
 # => [{:id=>1}, {:id=>2}]
{% endhighlight %}

<hr class="divider">
### map (column=nil, &block)

Maps column values for each record in the dataset (if a column name is given), or performs the stock mapping functionality of Enumerable otherwise. Raises an Error if both an argument and block are given.

{% highlight ruby %}
DB[:table].map(:id) # SELECT * FROM table
 # => [1, 2, 3, ...]

DB[:table].map{|r| r[:id] * 2} # SELECT * FROM table
 # => [2, 4, 6, ...]
{% endhighlight %}

You can also provide an array of column names:

{% highlight ruby %}
DB[:table].map([:id, :name])   #=> SELECT * FROM table
 # => [[1, 'A'], [2, 'B'], [3, 'C'], ...]
{% endhighlight %}


<hr class="divider">
### max (column=Sequel.virtual_row(&Proc.new))

Returns the maximum value for the given column/expression. Uses a virtual row block if no argument is given.

{% highlight ruby %}
DB[:table].max(:id) # SELECT max(id) FROM table LIMIT 1
 # => 10
DB[:table].max{function(column)} # SELECT max(function(column)) FROM table LIMIT 1
 # => 7
{% endhighlight %}

<hr class="divider">
### min (column=Sequel.virtual_row(&Proc.new))

Returns the minimum value for the given column/expression. Uses a virtual row block if no argument is given.

{% highlight ruby %}
DB[:table].min(:id) # SELECT min(id) FROM table LIMIT 1
 # => 1
DB[:table].min{function(column)} # SELECT min(function(column)) FROM table LIMIT 1
 # => 0
{% endhighlight %}

<hr class="divider">
### multi_insert (hashes, opts=OPTS)

This is a front end for import that allows you to submit an array of hashes instead of arrays of columns and values:

{% highlight ruby %}
DB[:table].multi_insert([{:x => 1}, {:x => 2}])
 # INSERT INTO table (x) VALUES (1)
 # INSERT INTO table (x) VALUES (2)
{% endhighlight %}

Be aware that all hashes should have the same keys if you use this calling method, otherwise some columns could be missed or set to null instead of to default values.

This respects the same options as import.

<hr class="divider">
### paged_each (opts=OPTS)

Yields each row in the dataset, but interally uses multiple queries as needed to process the entire result set without keeping all rows in the dataset in memory, even if the underlying driver buffers all query results in memory.

Because this uses multiple queries internally, in order to remain consistent, it also uses a transaction internally. Additionally, to work correctly, the dataset must have unambiguous order. Using an ambiguous order can result in an infinite loop, as well as subtler bugs such as yielding duplicate rows or rows being skipped.

Sequel checks that the datasets using this method have an order, but it cannot ensure that the order is unambiguous.

| **Options:** ||
| --- | --- |
| :rows_per_fetch	| The number of rows to fetch per query. Defaults to 1000.
| :strategy	| The strategy to use for paging of results. By default this is :offset, for using an approach with a limit and offset for every page. This can be set to :filter, which uses a limit and a filter that excludes rows from previous pages. In order for this strategy to work, you must be selecting the columns you are ordering by, and non of the columns can contain NULLs. Note that some Sequel adapters have optimised implementations that will use cursors or streaming regardless of the :strategy option used.
| :filter_values	| If the :strategy=>:filter option is used, this option should be a proc that accepts the last retrieved row for the previous page and an array of ORDER BY expressions, and returns an array of values relating to those expressions for the last retrieved row. You will need to use this option if your ORDER BY expressions are not simple columns, if they contain qualified identifiers that would be ambiguous unqualified, if they contain any identifiers that are aliased in SELECT, and potentially other cases.

**Examples:**

{% highlight ruby %}
DB[:table].order(:id).paged_each{|row| }
 # SELECT * FROM table ORDER BY id LIMIT 1000
 # SELECT * FROM table ORDER BY id LIMIT 1000 OFFSET 1000
 # ...

DB[:table].order(:id).paged_each(:rows_per_fetch=>100){|row| }
 # SELECT * FROM table ORDER BY id LIMIT 100
 # SELECT * FROM table ORDER BY id LIMIT 100 OFFSET 100
 # ...

DB[:table].order(:id).paged_each(:strategy=>:filter){|row| }
 # SELECT * FROM table ORDER BY id LIMIT 1000
 # SELECT * FROM table WHERE id > 1001 ORDER BY id LIMIT 1000
 # ...

DB[:table].order(:table__id).paged_each(:strategy=>:filter,
  :filter_values=>proc{|row, exprs| [row[:id]]}){|row| }
 # SELECT * FROM table ORDER BY id LIMIT 1000
 # SELECT * FROM table WHERE id > 1001 ORDER BY id LIMIT 1000
 # ...
{% endhighlight %}

<hr class="divider">
### range (column=Sequel.virtual_row(&Proc.new))

Returns a Range instance made from the minimum and maximum values for the given column/expression. Uses a virtual row block if no argument is given.

{% highlight ruby %}
DB[:table].range(:id) # SELECT max(id) AS v1, min(id) AS v2 FROM table LIMIT 1
 # => 1..10
DB[:table].interval{function(column)} # SELECT max(function(column)) AS v1, min(function(column)) AS v2 FROM table LIMIT 1
 # => 0..7
{% endhighlight %}

<hr class="divider">
### select_hash (key_column, value_column)

Returns a hash with key_column values as keys and value_column values as values. Similar to #to_hash, but only selects the columns given.

{% highlight ruby %}
DB[:table].select_hash(:id, :name) # SELECT id, name FROM table
 # => {1=>'a', 2=>'b', ...}
{% endhighlight %}

You can also provide an array of column names for either the key_column, the value column, or both:

{% highlight ruby %}
DB[:table].select_hash([:id, :foo], [:name, :bar]) # SELECT * FROM table
 # {[1, 3]=>['a', 'c'], [2, 4]=>['b', 'd'], ...}
{% endhighlight %}

When using this method, you must be sure that each expression has an alias that Sequel can determine. Usually you can do this by calling the as method on the expression and providing an alias.

<hr class="divider">

### select_hash_groups (key_column, value_column)

Returns a hash with key_column values as keys and an array of value_column values. Similar to #to_hash_groups, but only selects the columns given.

{% highlight ruby %}
DB[:table].select_hash_groups(:name, :id) # SELECT id, name FROM table
 # => {'a'=>[1, 4, ...], 'b'=>[2, ...], ...}
{% endhighlight %}

You can also provide an array of column names for either the key_column, the value column, or both:

{% highlight ruby %}
DB[:table].select_hash_groups([:first, :middle], [:last, :id]) # SELECT * FROM table
 # {['a', 'b']=>[['c', 1], ['d', 2], ...], ...}
{% endhighlight %}

When using this method, you must be sure that each expression has an alias that Sequel can determine. Usually you can do this by calling the as method on the expression and providing an alias.

<hr class="divider">
### select_map (column=nil, &block)

Selects the column given (either as an argument or as a block), and returns an array of all values of that column in the dataset. If you give a block argument that returns an array with multiple entries, the contents of the resulting array are undefined. Raises an Error if called with both an argument and a block.

{% highlight ruby %}
DB[:table].select_map(:id) # SELECT id FROM table
 # => [3, 5, 8, 1, ...]

DB[:table].select_map{id * 2} # SELECT (id * 2) FROM table
 # => [6, 10, 16, 2, ...]
{% endhighlight %}
 
You can also provide an array of column names:

{% highlight ruby %}
DB[:table].select_map([:id, :name]) # SELECT id, name FROM table
 # => [[1, 'A'], [2, 'B'], [3, 'C'], ...]
{% endhighlight %}

If you provide an array of expressions, you must be sure that each entry in the array has an alias that Sequel can determine. Usually you can do this by calling the as method on the expression and providing an alias.

<hr class="divider">
### select_order_map (column=nil, &block)

The same as #select_map, but in addition orders the array by the column.

{% highlight ruby %}
DB[:table].select_order_map(:id) # SELECT id FROM table ORDER BY id
 # => [1, 2, 3, 4, ...]

DB[:table].select_order_map{id * 2} # SELECT (id * 2) FROM table ORDER BY (id * 2)
 # => [2, 4, 6, 8, ...]
{% endhighlight %}

You can also provide an array of column names:

{% highlight ruby %}
DB[:table].select_order_map([:id, :name]) # SELECT id, name FROM table ORDER BY id, name
 # => [[1, 'A'], [2, 'B'], [3, 'C'], ...]
{% endhighlight %}

If you provide an array of expressions, you must be sure that each entry in the array has an alias that Sequel can determine. Usually you can do this by calling the as method on the expression and providing an alias.

<hr class="divider">

### single_record ()

Returns the first record in the dataset, or nil if the dataset has no records. Users should probably use first instead of this method.

<hr class="divider">

### single_value ()

Returns the first value of the first record in the dataset. Returns nil if dataset is empty. Users should generally use get instead of this method.

<hr class="divider">
### sum (column=Sequel.virtual_row(&Proc.new))

Returns the sum for the given column/expression. Uses a virtual row block if no column is given.

{% highlight ruby %}
DB[:table].sum(:id)  #=> SELECT sum(id) FROM table LIMIT 1
 # => 55
DB[:table].sum{function(column)}   #=> SELECT sum(function(column)) FROM table LIMIT 1
 # => 10
{% endhighlight %}

<hr class="divider">
### to_hash (key_column, value_column = nil)

Returns a hash with one column used as key and another used as value. If rows have duplicate values for the key column, the latter row(s) will overwrite the value of the previous row(s). If the value_column is not given or nil, uses the entire hash as the value.

{% highlight ruby %}
DB[:table].to_hash(:id, :name) # SELECT * FROM table
 # {1=>'Jim', 2=>'Bob', ...}

DB[:table].to_hash(:id) # SELECT * FROM table
 # {1=>{:id=>1, :name=>'Jim'}, 2=>{:id=>2, :name=>'Bob'}, ...}
{% endhighlight %}

You can also provide an array of column names for either the key_column, the value column, or both:

{% highlight ruby %}
DB[:table].to_hash([:id, :foo], [:name, :bar]) # SELECT * FROM table
 # {[1, 3]=>['Jim', 'bo'], [2, 4]=>['Bob', 'be'], ...}

DB[:table].to_hash([:id, :name]) # SELECT * FROM table
 # {[1, 'Jim']=>{:id=>1, :name=>'Jim'}, [2, 'Bob'=>{:id=>2, :name=>'Bob'}, ...}
{% endhighlight %}


<hr class="divider">
### to_hash_groups (key_column, value_column = nil)

Returns a hash with one column used as key and the values being an array of column values. If the value_column is not given or nil, uses the entire hash as the value.

{% highlight ruby %}
DB[:table].to_hash_groups(:name, :id) # SELECT * FROM table
 # {'Jim'=>[1, 4, 16, ...], 'Bob'=>[2], ...}

DB[:table].to_hash_groups(:name) # SELECT * FROM table
 # {'Jim'=>[{:id=>1, :name=>'Jim'}, {:id=>4, :name=>'Jim'}, ...], 'Bob'=>[{:id=>2, :name=>'Bob'}], ...}
{% endhighlight %}

You can also provide an array of column names for either the key_column, the value column, or both:

{% highlight ruby %}
DB[:table].to_hash_groups([:first, :middle], [:last, :id]) # SELECT * FROM table
 # {['Jim', 'Bob']=>[['Smith', 1], ['Jackson', 4], ...], ...}

DB[:table].to_hash_groups([:first, :middle]) # SELECT * FROM table
 # {['Jim', 'Bob']=>[{:id=>1, :first=>'Jim', :middle=>'Bob', :last=>'Smith'}, ...], ...}
{% endhighlight %}

### truncate ()

Truncates the dataset. Returns nil.

{% highlight ruby %}
DB[:table].truncate # TRUNCATE table # => nil
{% endhighlight %}

<hr class="divider">
### update (values=OPTS, &block)

Updates values for the dataset. The returned value is generally the number of rows updated, but that is adapter dependent. values should a hash where the keys are columns to set and values are the values to which to set the columns.

{% highlight ruby %}
DB[:table].update(:x=>nil) # UPDATE table SET x = NULL
 # => 10

DB[:table].update(:x=>:x+1, :y=>0) # UPDATE table SET x = (x + 1), y = 0
 # => 10
{% endhighlight %}

<hr class="divider">
### with_sql_all (sql, &block)

Run the given SQL and return an array of all rows. If a block is given, each row is yielded to the block after all rows are loaded. See with_sql_each.

<hr class="divider">
### with_sql_delete (sql)

Execute the given SQL and return the number of rows deleted. This exists solely as an optimization, replacing #with_sql(sql).delete. It's significantly faster as it does not require cloning the current dataset.

<hr class="divider">

### with_sql_each (sql)

Run the given SQL and yield each returned row to the block.

This method should not be called on a shared dataset if the columns selected in the given SQL do not match the columns in the receiver.

<hr class="divider">
### with_sql_first (sql)

Run the given SQL and return the first row, or nil if no rows were returned. See with_sql_each.

<hr class="divider">
### with_sql_insert (sql)

Execute the given SQL and (on most databases) return the primary key of the inserted row.

<hr class="divider">
### with_sql_single_value (sql)

Run the given SQL and return the first value in the first row, or nil if no rows were returned. For this to make sense, the SQL given should select only a single value. See with_sql_each.


## Protected Instance methods


###_import (columns, values, opts)

Internals of import. If primary key values are requested, use separate insert commands for each row. Otherwise, call multi_insert_sql and execute each statement it gives separately.


### _select_map_multiple (ret_cols)

Return an array of arrays of values given by the symbols in ret_cols.


<hr class="divider">

### _select_map_single ()

Returns an array of the first value in each row.

---

## User Methods relating to SQL Creation

## Public Instance methods

<hr class="divider">

### exists ()
Returns an EXISTS clause for the dataset as an SQL::PlaceholderLiteralString.

{% highlight ruby %}
DB.select(1).where(DB[:items].exists)
 # SELECT 1 WHERE (EXISTS (SELECT * FROM items))
{% endhighlight %}


<hr class="divider">
###  insert_sql (*values)
Returns an INSERT SQL query string. See insert.

{% highlight ruby %}
DB[:items].insert_sql(:a=>1)
 # => "INSERT INTO items (a) VALUES (1)"
{% endhighlight %}

<hr class="divider">

### literal_append (sql, v)

Append a literal representation of a value to the given SQL string.

If an unsupported object is given, an Error is raised.

<hr class="divider">

### multi_insert_sql (columns, values)

Returns an array of insert statements for inserting multiple records. This method is used by multi_insert to format insert statements and expects a keys array and and an array of value arrays.

This method should be overridden by descendants if the support inserting multiple records in a single SQL statement.

<hr class="divider">
### sql ()

Same as `select_sql`, not aliased directly to make subclassing simpler.

<hr class="divider">
### truncate_sql ()

Returns a TRUNCATE SQL query string. See truncate

{% highlight ruby %}
DB[:items].truncate_sql     # => 'TRUNCATE items'
{% endhighlight %}


<hr class="divider">
### update_sql (values = OPTS)

Formats an `UPDATE` statement using the given values. See update.

{% highlight ruby %}
DB[:items].update_sql(:price => 100, :category => 'software')
 # => "UPDATE items SET price = 100, category = 'software'
{% endhighlight %}

Raises an Error if the dataset is grouped or includes more than one table.



--- 





/EOF