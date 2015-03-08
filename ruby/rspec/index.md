---
title: [Ruby, "RSpec - Behaviour Driven Development"]
layout: default
asset_path: ../..

---

# {{ page.title | join: " / " }}

----

<div id="toc"></div>

---

## Sub-pages

### [RSpec-Core](/ruby/rspec/rspec-core.html)

### [RSpec-Expectations](/ruby/rspec/rspec-expectations.html)

### [RSpec-Mocks](/ruby/rspec/rspec-mocks.html)

### [RSpec-CollectionMatchers](/ruby/rspec/rspec-collection_matchers.html)

### [RSpec-HTML-Matchers](/ruby/rspec/rspec-html-matchers.html)


## Introduction to RSpec

Behaviour Driven Development for Ruby

### Description

`rspec` is a meta-gem, which depends on the [rspec-core](/rspec-core.html), [rspec-expectations](/rspec-expectations.html) and [rspec-mocks](/rspec-mocks.html) gems. Each of these can be installed separately and activated in isolation with the `gem` command.

Conversely, if you like RSpec's approach to declaring example groups and examples (`describe` and `it`) but prefer Test::Unit assertions and mocha, rr or flexmock for mocking, you'll be able to do that without having to load the
components of rspec that you're not using. 

### Documentation

#### rspec-core

* [Cucumber features](http://relishapp.com/rspec/rspec-core)
* [RDoc](http://rubydoc.info/gems/rspec-core/frames)

#### rspec-expectations

* [Cucumber features](http://relishapp.com/rspec/rspec-expectations)
* [RDoc](http://rubydoc.info/gems/rspec-expectations/frames)

#### rspec-mocks

* [Cucumber features](http://relishapp.com/rspec/rspec-mocks)
* [RDoc](http://rubydoc.info/gems/rspec-mocks/frames)

### Install

    gem install rspec

### Contribute

* [http://github.com/rspec/rspec-dev](http://github.com/rspec/rspec-dev)

### See Also

* [RSpec](/ruby/rspec/index.html) or [http://github.com/rspec/rspec](http://github.com/rspec/rspec)
* [RSpec::Core](/ruby/rspec/rspec-core.html) or [http://github.com/rspec/rspec-core](http://github.com/rspec/rspec-core)
* [RSpec::Expectations](/ruby/rspec/rspec-expectations.html) or [http://github.com/rspec/rspec-expectations](http://github.com/rspec/rspec-expectations)
* [RSpec::Mocks](/ruby/rspec/rspec-mocks.html) or [http://github.com/rspec/rspec-mocks](http://github.com/rspec/rspec-mocks)
* [RSpec::CollectionMatchers](/ruby/rspec/rspec-collection_matchers.html) or [http://github.com/rspec/rspec-collection_matchers](https://github.com/rspec/rspec-collection_matchers)
* [RSpec::HMTMLMatchers](/ruby/rspec/rspec-html-matchers.html) or [https://github.com/kucaahbe/rspec-html-matchers](https://github.com/kucaahbe/rspec-html-matchers)

