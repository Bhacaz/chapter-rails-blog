---
layout: default
title: 19 novembre 2020
parent: 2020
grand_parent: Réunion
nav_order: 21
---

# 19 novembre 2020

## Tour de table

## Save and associations

https://medium.com/@ynnadkrap/creating-activerecord-associations-d40bbdaba574

```ruby
class CreateCompaniesAndOffices < ActiveRecord::Migration[5.2]
  def change
    create_table :companies do |t|

      t.timestamps
    end

    create_table :offices do |t|
      t.references :company
      t.timestamps
    end
  end
end

class Company < ApplicationRecord
  has_one :office
end

class Office < ApplicationRecord
  belongs_to :company, optional: true
end
```


```ruby
o = Office.create!
c = Company.new(office: o)
c.persisted? # => false
o.save!
c.persisted? # => true
```

## [Rails 6.1.rc](https://weblog.rubyonrails.org/releases/)

### `destroy_async`

```ruby
class Author < ApplicationRecord
  has_many :books, dependent: :destroy_async
end
```

[Source](https://blog.saeloun.com/2020/11/18/rails-6.1-adds-support-for-destroying-dependent-associations-in-the-background.html)

## [The Best Ruby Blogs](https://dev.to/karllhughes/the-best-ruby-blogs-49h7)

## RubyMine Copy reference

Une des fonctionnalité que j'utilise le plus pour copy/paste des classes ou partager des lignes dans Slack.

Curseur sur une méthode/class/constante

![image](https://user-images.githubusercontent.com/7858787/99608810-a4d68c00-29dc-11eb-943b-ecd4992fc125.png)
 
```ruby
Absence#slots_by_category
```

![image](https://user-images.githubusercontent.com/7858787/99609126-4fe74580-29dd-11eb-8a46-2eab22e78c21.png)

```ruby
V2::Group::Accounts::AccountsApi
```

Curseur sur une ligne _random_

![image](https://user-images.githubusercontent.com/7858787/99608931-e5360a00-29dc-11eb-83e1-46f95c903e7b.png)

```text
app/models/common/absence.rb:49
```
