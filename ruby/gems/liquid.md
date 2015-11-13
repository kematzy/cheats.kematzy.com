---
title: [Ruby, Liquid]
layout: default
asset_path: ..

---

# {{ page.title | join: " / " }}



There are two types of markup in Liquid: Output and Tag.

* Output markup (which may resolve to text) is surrounded by


{% highlight liquid %}
{% raw %}
{{ matched pairs of curly brackets (ie, braces) }}
{% endraw %}
{% endhighlight %}

* Tag markup (which cannot resolve to text) is surrounded by

{% highlight liquid %}
{% raw %}
  {% matched pairs of curly brackets and percent signs %}
{% endraw %}
{% endhighlight %}



---

## Output

Here is a simple example of Output:

{% highlight liquid %}
{% raw %}
Hello {{name}}
Hello {{user.name}}
Hello {{ 'tobi' }}
{% endraw %}
{% endhighlight %}

<a name="filters"></a>

### Advanced output: Filters

Output markup takes filters.  Filters are simple methods.  The first parameter
is always the output of the left side of the filter.  The return value of the
filter will be the new left value when the next filter is run.  When there are
no more filters, the template will receive the resulting string.

{% highlight liquid %}
{% raw %}
Hello {{ 'tobi' | upcase }}
Hello tobi has {{ 'tobi' | size }} letters!
Hello {{ '*tobi*' | textilize | upcase }}
{% endraw %}
{% endhighlight %}

### Standard Filters

* `date` - reformat a date ([syntax reference](http://docs.shopify.com/themes/liquid-documentation/filters/additional-filters#date))

* `capitalize` - capitalize words in the input sentence

* `downcase` - convert an input string to lowercase

* `upcase` - convert an input string to uppercase

* `first` - get the first element of the passed in array

* `last` - get the last element of the passed in array

* `join` - join elements of the array with certain character between them

* `sort` - sort elements of the array

* `map` - map/collect an array on a given property

* `size` - return the size of an array or string

* `escape` - escape a string

* `escape_once` - returns an escaped version of html without affecting existing escaped entities

* `strip_html` - strip html from string

* `strip_newlines` - strip all newlines (\n) from string

* `newline_to_br` - replace each newline (\n) with html break

* `replace` - replace each occurrence *e.g.* `{{ 'foofoo' | replace:'foo','bar' }} #=> 'barbar'`

* `replace_first` - replace the first occurrence *e.g.* `{{ 'barbar' | replace_first:'bar','foo' }} #=> 'foobar'`

* `remove` - remove each occurrence *e.g.* `{{ 'foobarfoobar' | remove:'foo' }} #=> 'barbar'`

* `remove_first` - remove the first occurrence *e.g.* `{{ 'barbar' | remove_first:'bar' }} #=> 'bar'`

* `truncate` - truncate a string down to x characters. It also accepts a second parameter that will append to the string *e.g.* `{{ 'foobarfoobar' | truncate: 5, '.' }} #=> 'foob.'`

* `truncatewords` - truncate a string down to x words

* `prepend` - prepend a string *e.g.* `{{ 'bar' | prepend:'foo' }} #=> 'foobar'`

* `append` - append a string *e.g.* `{{ 'foo' | append:'bar' }} #=> 'foobar'`

* `slice` - slice a string. Takes an offset and length, *e.g.* `{{ "hello" | slice: -3, 3 }} #=> llo`

* `minus` - subtraction *e.g.*  `{{ 4 | minus:2 }} #=> 2`

* `plus` - addition *e.g.*  `{{ '1' | plus:'1' }} #=> '11'`, `{{ 1 | plus:1 }} #=> 2`

* `times` - multiplication  *e.g* `{{ 5 | times:4 }} #=> 20`

* `divided_by` - integer division *e.g.* `{{ 10 | divided_by:3 }} #=> 3`

* `split` - split a string on a matching pattern *e.g.* `{{ "a~b" | split:"~" }} #=> ['a','b']`

* `modulo` - remainder, *e.g.* `{{ 3 | modulo:2 }} #=> 1`

## Tags

Tags are used for the logic in your template. New tags are very easy to code, so I hope to get many contributions to the standard tag library after releasing this code.

Here is a list of currently supported tags:

* **assign** - Assigns some value to a variable

* **capture** - Block tag that captures text into a variable

* **case** - Block tag, its the standard case...when block

* **comment** - Block tag, comments out the text in the block

* **cycle** - Cycle is usually used within a loop to alternate between values, like colors or DOM classes.

* **for** - For loop

* **if** - Standard if/else block

* **include** - Includes another template; useful for partials

* **raw** - temporarily disable tag processing to avoid syntax conflicts.

* **unless** - Mirror of if statement

### Comments

Comment is the simplest tag.  It just swallows content.

{% highlight liquid %}
{% raw %}
We made 1 million dollars {% comment %} in losses {% endcomment %} this year
{% endraw %}
{% endhighlight %}

### Raw



Raw temporarily disables tag processing.
This is useful for generating content (eg, Mustache, Handlebars) which uses conflicting syntax.

{% highlight liquid %}
{% raw %}
  {{% raw %}}
    In Handlebars, {{ this }} will be HTML-escaped, but {{{ that }}} will not.
{{% endraw %}}
{% endraw %}
{% endhighlight %}


### If / Else

`if / else` should be well-known from any other programming language.
Liquid allows you to write simple expressions in the `if` or `unless` (and
optionally, `elsif` and `else`) clause:

{% highlight liquid %}
{% raw %}
{% if user %}
  Hello {{ user.name }}
{% endif %}
{% endraw %}
{% endhighlight %}


{% highlight liquid %}
{% raw %}
 # Same as above
{% if user != null %}
  Hello {{ user.name }}
{% endif %}
{% endraw %}
{% endhighlight %}


{% highlight liquid %}
{% raw %}
{% if user.name == 'tobi' %}
  Hello tobi
{% elsif user.name == 'bob' %}
  Hello bob
{% endif %}
{% endraw %}
{% endhighlight %}



{% highlight liquid %}
{% raw %}
{% if user.name == 'tobi' or user.name == 'bob' %}
  Hello tobi or bob
{% endif %}
{% endraw %}
{% endhighlight %}


{% highlight liquid %}
{% raw %}
{% if user.name == 'bob' and user.age > 45 %}
  Hello old bob
{% endif %}
{% endraw %}
{% endhighlight %}


{% highlight liquid %}
{% raw %}
{% if user.name != 'tobi' %}
  Hello non-tobi
{% endif %}
{% endraw %}
{% endhighlight %}


{% highlight liquid %}
{% raw %}
 # Same as above
{% unless user.name == 'tobi' %}
  Hello non-tobi
{% endunless %}
{% endraw %}
{% endhighlight %}



{% highlight liquid %}
{% raw %}
 # Check for the size of an array
{% if user.payments == empty %}
   you never paid !
{% endif %}

{% if user.payments.size > 0  %}
   you paid !
{% endif %}
{% endraw %}
{% endhighlight %}



{% highlight liquid %}
{% raw %}
{% if user.age > 18 %}
   Login here
{% else %}
   Sorry, you are too young
{% endif %}
{% endraw %}
{% endhighlight %}


{% highlight liquid %}
{% raw %}
 # array = 1,2,3
{% if array contains 2 %}
   array includes 2
{% endif %}
{% endraw %}
{% endhighlight %}




{% highlight liquid %}
{% raw %}
 # string = 'hello world'
{% if string contains 'hello' %}
   string includes 'hello'
{% endif %}
{% endraw %}
{% endhighlight %}





### Case Statement

If you need more conditions, you can use the `case` statement:

{% highlight liquid %}
{% raw %}
{% case condition %}
{% when 1 %}
hit 1
{% when 2 or 3 %}
hit 2 or 3
{% else %}
... else ...
{% endcase %}
{% endraw %}
{% endhighlight %}


*Example:*

{% highlight liquid %}
{% raw %}
{% case template %}

{% when 'label' %}
     // {{ label.title }}
{% when 'product' %}
     // {{ product.vendor | link_to_vendor }} / {{ product.title }}
{% else %}
     // {{page_title}}
{% endcase %}
{% endraw %}
{% endhighlight %}




### Cycle

Often you have to alternate between different colors or similar tasks.  Liquid
has built-in support for such operations, using the `cycle` tag.

{% highlight liquid %}
{% raw %}
{% cycle 'one', 'two', 'three' %}
{% cycle 'one', 'two', 'three' %}
{% cycle 'one', 'two', 'three' %}
{% cycle 'one', 'two', 'three' %}
{% endraw %}
{% endhighlight %}


will result in


```
one
two
three
one
```

If no name is supplied for the cycle group, then it's assumed that multiple calls with the same parameters are one group.

If you want to have total control over cycle groups, you can optionally specify the name of the group.  This can even be a variable.

{% highlight liquid %}
{% raw %}
{% cycle 'group 1': 'one', 'two', 'three' %}
{% cycle 'group 1': 'one', 'two', 'three' %}
{% cycle 'group 2': 'one', 'two', 'three' %}
{% cycle 'group 2': 'one', 'two', 'three' %}
{% endraw %}
{% endhighlight %}

will result in

{% highlight ruby %}
one
two
one
two
{% endhighlight %}



### For loops

Liquid allows `for` loops over collections:

{% highlight liquid %}
{% raw %}
{% for item in array %}
  {{ item }}
{% endfor %}
{% endraw %}
{% endhighlight %}


When iterating a hash, `item[0]` contains the key, and `item[1]` contains the value:


{% highlight liquid %}
{% raw %}
{% for item in hash %}
  {{ item[0] }}: {{ item[1] }}
{% endfor %}
{% endraw %}
{% endhighlight %}

During every `for` loop, the following helper variables are available for extra styling needs:


{% highlight liquid %}
{% raw %}
forloop.length      # => length of the entire for loop
forloop.index       # => index of the current iteration
forloop.index0      # => index of the current iteration (zero based)
forloop.rindex      # => how many items are still left?
forloop.rindex0     # => how many items are still left? (zero based)
forloop.first       # => is this the first iteration?
forloop.last        # => is this the last iteration?
{% endraw %}
{% endhighlight %}


There are several attributes you can use to influence which items you receive in your loop

`limit:int` lets you restrict how many items you get.
`offset:int` lets you start the collection with the nth item.


{% highlight liquid %}
{% raw %}
 # array = [1,2,3,4,5,6]
{% for item in array limit:2 offset:2 %}
  {{ item }}
{% endfor %}
 # results in 3,4
{% endraw %}
{% endhighlight %}



Reversing the loop


{% highlight liquid %}
{% raw %}
{% for item in collection reversed %} {{item}} {% endfor %}
{% endraw %}
{% endhighlight %}



Instead of looping over an existing collection, you can define a range of numbers to loop through.  The range can be defined by both literal and variable numbers:


{% highlight liquid %}
{% raw %}
 # if item.quantity is 4...
{% for i in (1..item.quantity) %}
  {{ i }}
{% endfor %}
 # results in 1,2,3,4
{% endraw %}
{% endhighlight %}



### Variable Assignment

You can store data in your own variables, to be used in output or other tags as
desired.  The simplest way to create a variable is with the `assign` tag, which
has a pretty straightforward syntax:


{% highlight liquid %}
{% raw %}
{% assign name = 'freestyle' %}

{% for t in collections.tags %}{% if t == name %}
  <p>Freestyle!</p>
{% endif %}{% endfor %}
{% endraw %}
{% endhighlight %}


Another way of doing this would be to assign `true / false` values to the variable:


{% highlight liquid %}
{% raw %}
{% assign freestyle = false %}

{% for t in collections.tags %}{% if t == 'freestyle' %}
  {% assign freestyle = true %}
{% endif %}{% endfor %}

{% if freestyle %}
  <p>Freestyle!</p>
{% endif %}
{% endraw %}
{% endhighlight %}

If you want to combine a number of strings into a single string and save it to
a variable, you can do that with the `capture` tag. This tag is a block which
"captures" whatever is rendered inside it, then assigns the captured value to
the given variable instead of rendering it to the screen.


{% highlight liquid %}
{% raw %}
  {% capture attribute_name %}{{ item.title | handleize }}-{{ i }}-color{% endcapture %}

  <label for="{{ attribute_name }}">Color:</label>
  <select name="attributes[{{ attribute_name }}]" id="{{ attribute_name }}">
    <option value="red">Red</option>
    <option value="green">Green</option>
    <option value="blue">Blue</option>
  </select>
{% endraw %}
{% endhighlight %}



---


## Advanced Arrays in Shopify's Liquid

29 JULY 2014

Liquid is Shopify's templating language. It's great and packed with useful helpers. Unfortunately, it struggles when it comes to creating arrays (probably fair, it's a templating language, afterall).

Creating an array is a simple process, albeit not very intuitive:

{% highlight liquid %}
{% raw %}
{% assign myArray = "item-1|item-2|item-3" | split: "|" %}
{% endraw %}
{% endhighlight %}


We're assigning a string to a new variable called myArray. The array is created when we use the split filter. Now we can run a for loop on our array:

{% highlight liquid %}
{% raw %}
  {% for item in myArray %}
    {{ item }}
  {% endfor %}
{% endraw %}
{% endhighlight %}

Will output:


item-1
item-2
item-3

## Associative Arrays

Now things get a little complicated. Shopify has a great new feature for sorting collections. Instead of creating the dropdown manually, I wanted to create a simple for loop that looked like:

{% highlight liquid %}
{% raw %}
{% assign sortTitles = "Featured|$ Low to High|$ High to Low|A-Z|Z-A|Oldest to Newest|Newest to Oldest|Best Selling" | split: "|" %}
<select>
  {% for title in sortTitles %}
    <option value="{{ title | handle }}">{{ title }}</option>
  {% endfor %}
</select>
{% endraw %}
{% endhighlight %}

Unfortunately, {{ title | handle }} does not accurately correspond to the values we need to pass to the URL. I need to tie (or associate) the handle with a front-facing title that anyone could edit. Double array to the rescue!

{% highlight liquid %}
{% raw %}
{% assign sortHandles = "manual|price-ascending|price-descending|title-ascending|title-descending|created-ascending|created-descending|best-selling" | split: "|" %}
{% assign sortTitles = "Featured|$ Low to High|$ High to Low|A-Z|Z-A|Oldest to Newest|Newest to Oldest|Best Selling" | split: "|" %}

<select>
  {% for handle in sortHandles %}
    <option value="{{ handle }}">{{ sortTitles[forloop.index0] }}</option>
  {% endfor %}
</select>
{% endraw %}
{% endhighlight %}

We can use the current loop's index value and use it to get our title from the sortTitles array. You just need to make sure that if you reorder any of the items in one array you do the same reordering in the other one.

The final markup, if you're interested, looks like this:

{% highlight liquid %}
{% raw %}
{% for handle in sortHandles %}
  {% if collection.sort_by == blank and collection.default_sort_by == handle %}
    {% assign currentTitle = sortTitles[forloop.index0] %}
  {% elsif collection.sort_by == handle %}
    {% assign currentTitle = sortTitles[forloop.index0] %}
  {% endif %}
{% endfor %}

<label class="selected-text">Sort by: <strong>{{ currentTitle }}</strong></label>
<select>
  {% for handle in sortHandles %}
    {% if collection.sort_by == blank and collection.default_sort_by == handle %}
      <option value="{{handle}}" selected="selected">{{ sortTitles[forloop.index0] }}</option>
    {% elsif collection.sort_by == handle %}
      <option value="{{handle}}" selected="selected">{{ sortTitles[forloop.index0] }}</option>
    {% else %}
      <option value="{{handle}}">{{ sortTitles[forloop.index0] }}</option>
    {% endif %}
  {% endfor %}
</select>
{% endraw %}
{% endhighlight %}


There's quite a bit of JavaScript involved as well, but we'll save that for another post!
