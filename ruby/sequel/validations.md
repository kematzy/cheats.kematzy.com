---
title: [Ruby, Sequel ORM, Validations]
layout: default
asset_path: ../..

---

# {{ page.title | join: " / " }}


{% highlight ruby %}

{% endhighlight %}


## Overview



## Validations


{% highlight ruby %}
def validate
  super
  errors.add(:name, 'cannot be empty') if !name || name.empty?

  validates_presence [:title, :site]
  validates_unique :name
  validates_format /\Ahttps?:\/\/\\/, :website, :message=>'is not a valid URL'
  validates_includes %w(a b c), :type
  validates_integer :rating
  validates_numeric :number
  validates_type String, [:title, :description]

  validates_integer :rating  if new?

  # options: :message =>, :allow_nil =>, :allow_blank =>,
  #          :allow_missing =>,

  validates_exact_length 17, :isbn
  validates_min_length 3, :name
  validates_max_length 100, :name
  validates_length_range 3..100, :name

  # Setter override
  def filename=(name)
    @values[:filename] = name
  end
end
end

deal.errors

{% endhighlight %}


<br>
### - validates_presence(atts, opts=OPTS)
Check attribute value(s) is not considered `blank` by the database, but allow `false` values.

{% highlight ruby %}
 # validating single field
validates_presence(:title)

 # validating multiple fields
validates_presence [:title, :site]
{% endhighlight %}

<br>
### - validates_numeric (atts, opts=OPTS)
Check attribute value(s) string representation is a valid float.


{% highlight ruby %}
 # validating single field
validates_numeric(:price)

 # validating multiple fields
validates_numeric [:price, :discounted_price]
{% endhighlight %}

<br>
### - validates_exact_length (exact, atts, opts=OPTS)
Check that the attribute values are the given exact length.

{% highlight ruby %}
validates_exact_length 17, :isbn

validates_exact_length 2, :country_code
{% endhighlight %}


<br>
### - validates_format (with, atts, opts=OPTS)
Check the string representation of the attribute value(s) against the regular expression with.

{% highlight ruby %}
validates_format /\Ahttps?:\/\//, :website, :message=>'is not a valid URL'
{% endhighlight %}


<br>
### - validates_includes (set, atts, opts=OPTS)
Check attribute value(s) is included in the given set.

{% highlight ruby %}
validates_includes %w(a b c), :type
{% endhighlight %}


<br>
### - validates_integer (atts, opts=OPTS)
Check attribute value(s) string representation is a valid integer.

{% highlight ruby %}
validates_integer(:rating) #=> ":rating is not a number"

 # only validate value if new record
validates_integer :rating  if new?

{% endhighlight %}

<br>
### - validates_length_range (range, atts, opts=OPTS)
Check that the attribute values length is in the specified range.

{% highlight ruby %}
validates_length_range(3..100, :name)  #=> ":name is too short or too long"
{% endhighlight %}


<br>
### - validates_max_length (max, atts, opts=OPTS)
Check that the attribute values are not longer than the given max length.

Accepts a `:nil_message` option that is the error message to use when the value is `nil` instead of being too long.

{% highlight ruby %}
validates_max_length( 100, :name)  #=> ":name is longer than #{max} characters"
{% endhighlight %}

<br>
### - validates_min_length (min, atts, opts=OPTS)
Check that the attribute values are not shorter than the given min length.

{% highlight ruby %}
validates_min_length( 3, :name) #=>
{% endhighlight %}


<br>
### - validates_not_null (atts, opts=OPTS)
Check attribute value(s) are not` NULL/nil`.

{% highlight ruby %}
 # TODO: add example here...
{% endhighlight %}


<br>
### - validates_schema_types (atts=keys, opts=OPTS)
Validates for all of the model columns (or just the given columns) that the column value is an instance of the expected class based on the column's schema type.

{% highlight ruby %}
 # TODO: add example here...
{% endhighlight %}


<br>
### - validates_type (klass, atts, opts=OPTS)
Check if value is an instance of a class. If `klass` is an array, the value must be an instance of one of the classes in the array.

{% highlight ruby %}
validates_type String, [:title, :description]

 # value must one of the classes in the klass array
validates_type [String,Integer], [:title, :description, :id]
{% endhighlight %}


<br>
### - validates_unique (*atts)
Checks that there are no duplicate values in the database for the given attributes. Pass an array of fields instead of multiple fields to specify that the combination of fields must be unique, instead of that each field should have a unique value.

#### Note!!  - Important Syntax Versions

{% highlight ruby %}
 # validate grouping of fields (column1 and column2) together
validates_unique([:column1, :column2])

 # validates each field separately
validates_unique(:column1, :column2)
{% endhighlight %}


You can pass a block, which is yielded the dataset in which the columns must be unique. So if you are doing a soft delete of records, in which the name must be unique, but only for active records:

{% highlight ruby %}
validates_unique(:name){|ds| ds.filter(:active)}
{% endhighlight %}

You should **also add a unique index in the database**, as this suffers from a fairly obvious race condition.

This validation does not respect the `:allow_*` options that the other validations accept, since it can deal with a grouping of multiple attributes.

#### Possible Options:


* **:dataset** - The base dataset to use for the unique query, defaults to the model's dataset

* **:message** -  The message to use (default: 'is already taken')

* **:only_if_modified**	- Only check the uniqueness if the object is new or one of the columns has been modified.

* **:where** -  A callable object where call takes three arguments, a dataset, the current object, and an array of columns, and should return a modified dataset that is filtered to include only rows with the same values as the current object for each column in the array.


Case insensitive uniqueness validation on a database that is case sensitive by default, use:

{% highlight ruby %}
 #
validates_unique :column, :where=>(proc do |ds, obj, cols|
  ds.where(cols.map do |c|
    v = obj.send(c)
    v = v.downcase if v
    [Sequel.function(:lower, c), v]
  end)
end)

{% endhighlight %}


<br>
<br>
---

## Default Options


{% highlight ruby %}
DEFAULT_OPTIONS  =  {
  #
  :exact_length=>{
    :message=>lambda{|exact| "is not #{exact} characters"}
  },
  #  
  :format=>{
    :message=>lambda{|with| 'is invalid'}
  },
  #
  :includes=>{
    :message=>lambda{|set| "is not in range or set: #{set.inspect}"}
  },
  #
  :integer=>{
    :message=>lambda{"is not a number"}
  },
  #
  :length_range=>{
    :message=>lambda{|range| "is too short or too long"}
  },
  #
  :max_length=>{
    :message=>lambda{|max| "is longer than #{max} characters"},
    :nil_message=>lambda{"is not present"}
  },
  #
  :min_length=>{
    :message=>lambda{|min| "is shorter than #{min} characters"}
  },
  #
  :not_null=>{
    :message=>lambda{"is not present"}
  },
  #
  :numeric=>{
    :message=>lambda{"is not a number"}
  },
  #
  :type=>{
    :message=>lambda{|klass| klass.is_a?(Array) ? "is not a valid #{klass.join(" or ").downcase}" : "is not a valid #{klass.to_s.downcase}"}
  },
  #
  :presence=>{
    :message=>lambda{"is not present"}
  },
  #
  :unique=>{
    :message=>lambda{'is already taken'}
  }
}
{% endhighlight %}



Default validation options used by Sequel. Can be modified to change the error messages for all models (e.g. for internationalization), or to set certain default options for validations (e.g. `:allow_nil=>true` for all `validates_format`).
