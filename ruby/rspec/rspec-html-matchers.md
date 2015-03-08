---
title: [Ruby, RSpec, RSpec::HTMLMatchers]
layout: default
asset_path: ../..

---

# {{ page.title | join: " / " }}

----

<div id="toc"></div>

---

[Github Repo](https://github.com/kucaahbe/rspec-html-matchers)

## Introduction

[RSpec 3](https://github.com/rspec) matchers for testing your html (for [RSpec 2](https://www.relishapp.com/rspec/rspec-core/v/2-99/docs) use 0.5.x version).


### Goals

* for testing **complicated** html output, for simple matching consider use:
  * [assert_select](http://api.rubyonrails.org/classes/ActionDispatch/Assertions/SelectorAssertions.html#method-i-assert_select)
  * [matchers provided out of the box in rspec-rails](https://www.relishapp.com/rspec/rspec-rails/v/2-11/docs/view-specs/view-spec)
  * [matchers provided by capybara](http://rdoc.info/github/jnicklas/capybara/Capybara/Node/Matchers)
* developer-firendly output in error messages
* built on top of [nokogiri](nokogiri.org)
* has support for [capybara](https://github.com/jnicklas/capybara), see below
* syntax is similar to [have_tag](http://old.rspec.info/rails/writing/views.html) matcher from old-school rspec-rails, but with own syntactic sugar
* framework agnostic, as input should be String(or capybara's page, see below)

### Install

Add to your Gemfile in the `:test` group:

```ruby
gem 'rspec-html-matchers'
```

and somewhere in RSpec configuration:

```ruby
RSpec.configure do |config|
  config.include RSpecHtmlMatchers
end
```

or just in you spec(s):

```ruby
describe "my view spec" do
  include RSpecHtmlMatchers

  it "has tags" do
    expect(rendered).to have_tag('div')
  end

end
```

Cucumber configuration:

```ruby
World RSpecHtmlMatchers
```

as this gem requires **nokogiri**, here [instructions for installing it](http://nokogiri.org/tutorials/installing_nokogiri.html).

### Usage

so perhaps your code produces following output:

```html
<h1>Simple Form</h1>
<form action="/users" method="post">
<p>
  <input type="email" name="user[email]" />
</p>
<p>
  <input type="submit" id="special_submit" />
</p>
</form>
```

so you test it with following:

```ruby
expect(rendered).to have_tag('form', :with => { :action => '/users', :method => 'post' }) do
  with_tag "input", :with => { :name => "user[email]", :type => 'email' }
  with_tag "input#special_submit", :count => 1
  without_tag "h1", :text => 'unneeded tag'
  without_tag "p",  :text => /content/i
end
```

Example about should be self-descriptive, but if not refer to [have_tag](http://rdoc.info/github/kucaahbe/rspec-html-matchers/RSpec/Matchers:have_tag) documentation

Input could be any html string. Let's take a look at these examples:

#### matching tags by css:

Simple examples:

```ruby
expect('<p class="qwe rty" id="qwerty">Paragraph</p>').to have_tag('p')
expect('<p class="qwe rty" id="qwerty">Paragraph</p>').to have_tag(:p)
expect('<p class="qwe rty" id="qwerty">Paragraph</p>').to have_tag('p#qwerty')
expect('<p class="qwe rty" id="qwerty">Paragraph</p>').to have_tag('p.qwe.rty')
```

More complicated examples:

```ruby
expect('<p class="qwe rty" id="qwerty"><strong>Para</strong>graph</p>').to have_tag('p strong')
expect('<p class="qwe rty" id="qwerty"><strong>Para</strong>graph</p>').to have_tag('p#qwerty strong')
expect('<p class="qwe rty" id="qwerty"><strong>Para</strong>graph</p>').to have_tag('p.qwe.rty strong')
```

or you can use another syntax for examples above

```ruby
expect('<p class="qwe rty" id="qwerty"><strong>Para</strong>graph</p>').to have_tag('p') do
  with_tag('strong')
end

expect('<p class="qwe rty" id="qwerty"><strong>Para</strong>graph</p>').to have_tag('p#qwerty') do
  with_tag('strong')
end

expect('<p class="qwe rty" id="qwerty"><strong>Para</strong>graph</p>').to have_tag('p.qwe.rty') do
  with_tag('strong')
end
```

#### Special case for classes matching:


```ruby
 # all of these are equivalent:
expect('<p class="qwe rty" id="qwerty">Paragraph</p>').to have_tag('p', :with => { :class => 'qwe rty' })
expect('<p class="qwe rty" id="qwerty">Paragraph</p>').to have_tag('p', :with => { :class => 'rty qwe' })
expect('<p class="qwe rty" id="qwerty">Paragraph</p>').to have_tag('p', :with => { :class => ['rty', 'qwe'] })
expect('<p class="qwe rty" id="qwerty">Paragraph</p>').to have_tag('p', :with => { :class => ['qwe', 'rty'] })
```

The same works with `:without`:

```ruby
 # all of these are equivalent:
expect('<p class="qwe rty" id="qwerty">Paragraph</p>').to have_tag('p', :without => { :class => 'qwe rty' })
expect('<p class="qwe rty" id="qwerty">Paragraph</p>').to have_tag('p', :without => { :class => 'rty qwe' })
expect('<p class="qwe rty" id="qwerty">Paragraph</p>').to have_tag('p', :without => { :class => ['rty', 'qwe'] })
expect('<p class="qwe rty" id="qwerty">Paragraph</p>').to have_tag('p', :without => { :class => ['qwe', 'rty'] })
```

### Content matching:

```ruby
expect('<p> Some content&nbsphere</p>').to have_tag('p', :text => ' Some content here')
```

or

```ruby
expect('<p> Some content&nbsphere</p>').to have_tag('p') do
  with_text ' Some content here'
end

expect('<p> Some content&nbsphere</p>').to have_tag('p', :text => /Some content here/)
```
or

```ruby
expect('<p> Some content&nbsphere</p>').to have_tag('p') do
  with_text /Some content here/
end

 # mymock.text == 'Some content here'
expect('<p> Some content&nbsphere</p>').to have_tag('p', :content => mymock.text)
```

or

```ruby
expect('<p> Some content&nbsphere</p>').to have_tag('p') do
  with_content mymock.text
end
```

### Usage with Capybara and Cucumber:

```ruby
expect(page).to have_tag( ... )
```

where `page` is an instance of Capybara::Session

### Shorthand matchers for Form inputs:

- [have\_form](http://rdoc.info/github/kucaahbe/rspec-html-matchers/master/RSpec/Matchers:have_form)
- [with\_checkbox](http://rdoc.info/github/kucaahbe/rspec-html-matchers/master/RSpec/Matchers:with_checkbox)
- [with\_email\_field](http://rdoc.info/github/kucaahbe/rspec-html-matchers/master/RSpec/Matchers:with_email_field)
- [with\_file\_field](http://rdoc.info/github/kucaahbe/rspec-html-matchers/master/RSpec/Matchers:with_file_field)
- [with\_hidden\_field](http://rdoc.info/github/kucaahbe/rspec-html-matchers/master/RSpec/Matchers:with_hidden_field)
- [with\_option](http://rdoc.info/github/kucaahbe/rspec-html-matchers/master/RSpec/Matchers:with_option)
- [with\_password_field](http://rdoc.info/github/kucaahbe/rspec-html-matchers/master/RSpec/Matchers:with_password_field)
- [with\_radio\_button](http://rdoc.info/github/kucaahbe/rspec-html-matchers/master/RSpec/Matchers:with_radio_button)
- [with\_button](http://rdoc.info/github/kucaahbe/rspec-html-matchers/master/RSpec/Matchers:with_button)
- [with\_select](http://rdoc.info/github/kucaahbe/rspec-html-matchers/master/RSpec/Matchers:with_select)
- [with\_submit](http://rdoc.info/github/kucaahbe/rspec-html-matchers/master/RSpec/Matchers:with_submit)
- [with\_text\_area](http://rdoc.info/github/kucaahbe/rspec-html-matchers/master/RSpec/Matchers:with_text_area)
- [with\_text\_field](http://rdoc.info/github/kucaahbe/rspec-html-matchers/master/RSpec/Matchers:with_text_field)
- [with\_url\_field](http://rdoc.info/github/kucaahbe/rspec-html-matchers/master/RSpec/Matchers:with_url_field)
- [with\_number\_field](http://rdoc.info/github/kucaahbe/rspec-html-matchers/master/RSpec/Matchers:with_number_field)
- [with\_range\_field](http://rdoc.info/github/kucaahbe/rspec-html-matchers/master/RSpec/Matchers:with_range_field)
- [with\_date\_field](http://rdoc.info/github/kucaahbe/rspec-html-matchers/master/RSpec/Matchers:with_date_field)

and of course you can use the `without_` matchers (see the documentation).


### More info

You can find more on [RubyDoc](http://rubydoc.info/github/kucaahbe/rspec-html-matchers/master/RSpec/Matchers), take a look at [have_tag](http://rdoc.info/github/kucaahbe/rspec-html-matchers/RSpec/Matchers#have_tag-instance_method) method.

Also, please read [CHANGELOG](https://github.com/kucaahbe/rspec-html-matchers/blob/master/CHANGELOG.md), it might be helpful.

-------------

### See Also

* [RSpec](/ruby/rspec/index.html) or [http://github.com/rspec/rspec](http://github.com/rspec/rspec)
* [RSpec::Core](/ruby/rspec/rspec-core.html) or [http://github.com/rspec/rspec-core](http://github.com/rspec/rspec-core)
* [RSpec::Expectations](/ruby/rspec/rspec-expectations.html) or [http://github.com/rspec/rspec-expectations](http://github.com/rspec/rspec-expectations)
* [RSpec::Mocks](/ruby/rspec/rspec-mocks.html) or [http://github.com/rspec/rspec-mocks](http://github.com/rspec/rspec-mocks)
* [RSpec::CollectionMatchers](/ruby/rspec/rspec-collection_matchers.html) or [http://github.com/rspec/rspec-collection_matchers](https://github.com/rspec/rspec-collection_matchers)
* [RSpec::HMTMLMatchers](/ruby/rspec/rspec-html-matchers.html) or [https://github.com/kucaahbe/rspec-html-matchers](https://github.com/kucaahbe/rspec-html-matchers)


