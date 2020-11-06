---
layout: default
title: 19 novembre 2020
parent: 2020
grand_parent: Réunion
nav_order: 21
---

# 19 novembre 2020

## Tour de table

## Auto save association

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
