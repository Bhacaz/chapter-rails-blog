---
layout: default
title: 8 octobre 2020
parent: 2020
grand_parent: Réunion
nav_order: 18
---

# 8 octobre 2020

## Tour de table

## [Ruby 3.0.0 preview 1](https://www.ruby-lang.org/en/news/2020/09/25/ruby-3-0-0-preview1-released/)

* Rightward assignment statement

```ruby
RUBY_VERSION # => 3.0.0

array = [1, 2, 3, 4]
array.select(&:odd?).map(&:to_s) => b
b # => ["1", "3"]
```

* Endless method definition

```ruby
RUBY_VERSION # => 3.0.0

def greating(name) = "Hello #{name}"
greating('Ruby') # => "Hello Ruby"
```

Et autres [NEWS](https://github.com/ruby/ruby/blob/v3_0_0_preview1/NEWS.md)

## Chewy et Elasticsearch 7 😢

## [Guard](https://github.com/guard/guard)

