---
layout: default
title: 21 novembre 2019
parent: Réunion 2019
grand_parent: Réunion
nav_order: 4
---

# 21 novembre 2019

## Tour de table

## Update des gems

6/39 (15%) 👏
* factory_bot_rails
* simplecov
* timecop
* webmock
* ar_transaction_changes
* groupdate

## [Suppression de legacy](https://docs.google.com/spreadsheets/d/1qB9MCutPvdpUjfq2lHprfgLjwvqclxX_IS4ImIHavO0/edit?usp=sharing)


## ~~protected_attributes~~ => strong_parameters

![enter image description here](https://res.cloudinary.com/practicaldev/image/fetch/s--itirl5W6--/c_limit,f_auto,fl_progressive,q_auto,w_880/https://thepracticaldev.s3.amazonaws.com/i/9qyd7fvs06wbjrabzowu.png)
[https://dev.to/hint/moderate-parameters-the-path-from-protected-attributes-to-strong-parameters-22il](https://dev.to/hint/moderate-parameters-the-path-from-protected-attributes-to-strong-parameters-22il)


## permitted_attributes
Est applicable partout
```ruby
class Account < ActiveRecord::Base  
  attr_accessible :last_name  
end  
  
# fail
@account.update! last_name: 'hello', kind_id: 1  
  
# pass
@account.update! kind_id: 1  
@account.last_name = 'hello'  
@account.save!  
```

## strong_parameters  
Est seulement applicable dans les controllers 🙏
```ruby
params = ActionController::Parameters.new(account: { last_name: 'hello', kind_id: 1 })

# fail
@account.update! params[:account]
  
# pass
@account.update! params.require(:account).permit(:last_name, :kind_id)  
@account.update! params[:account].to_h

# Ma solution
@account.update! params.require(:account).permit(*params[:account].keys)
```

## Comment ça marche

```ruby
# activemodel-4.2.11.1/lib/active_model/forbidden_attributes_protection.rb:20

def sanitize_for_mass_assignment(attributes)  
  if attributes.respond_to?(:permitted?) && !attributes.permitted?  
    raise ActiveModel::ForbiddenAttributesError  
  else  
  attributes  
  end  
end

{}.respond_to?(:permitted?) # => false
HashWithIndifferentAccess.new.respond_to?(:permitted?) # => false
ActionController::Parameters.new({}).respond_to?(:permitted?) # => true
```


## Callback `:on` vs `:if`

```ruby
class Territory < Lookup  
  after_commit :print, on: :update, if: :start_with_a?  
  
  def start_with_a?  
    puts "Called start_with_a?"  
    mnemonic.start_with?('A')  
  end  
  
  def print  
    puts mnemonic  
  end  
end
```
**create**
```ruby
>> Territory.create! mnemonic: 'A123', country_id: 2
Called start_with_a? # 😱
A123 {
            :id => 20,
      :mnemonic => "A123",
    :country_id => 2
}
```
**update**

```ruby
>> Territory.last.update! mnemonic: 'A1234'
Called start_with_a?
A1234
```

Fix [Rails 5.1.0](https://github.com/rails/rails/pull/28063/commits/c478c74c183c480398eab822f4265e06a9501d36)


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM4MjQ2NDk5NSwtMTQ5ODkxOTA2OCwtMT
I5MzkyMzQzMCwzODEyNjkwOTksLTExMDQ0Mzc1ODUsNjY0MzU5
NjY2LDI1NzA1NzU2Nyw3MzA5OTgxMTZdfQ==
-->