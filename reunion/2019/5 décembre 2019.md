---
layout: default
title: 5 décembre 2019
parent: Réunion 2019
grand_parent: Réunion
nav_order: 5
---

# 5 décembre 2019

## Tour de table

### Grape#route_params & permitted_params

```ruby  
resource :accounts do    
  route_params :id do    
    params do    
      optional :some_params, type: String, default: 'hello'    
    end    
    
    get do    
      puts permitted_params # => { some_params: 'hello' }    
    end    
  end  
end  
```

```ruby  
resource :accounts do  
  params do
   require :id, type: Integer 
  end
  
  route_params :id do
    params do
     optional :some_params, type: String, default: 'hello'
    end  
  
     get do 
       puts permitted_params # => { id: 123, some_params: 'hello' } 
     end
  end
end
```

## [Warning Hash#method_missing](https://github.com/petalmd/petalmd.rails/pull/5030)
Votre avis?

```ruby
>> {}.hello
'hello' is not a method, use [] 
(irb):2:in `irb_binding'
```

## benchmark-ips
Maintenant dans le Gemfile. Juste besoin du `require`
**Snippet**
```ruby
require 'benchmark/ips'  
  
Benchmark.ips do |x|  
  x.report('method_missing') { {}.hello }  
  x.report('[]') { {}[:hello] }  
  x.compare!  
end
```

**Result**
```
Warming up --------------------------------------
      method_missing   111.209k i/100ms
                  []   293.072k i/100ms
Calculating -------------------------------------
      method_missing      1.729M (± 4.0%) i/s -      8.674M in   5.024302s
                  []      9.148M (± 7.7%) i/s -     45.719M in   5.027372s

Comparison:
                  []:  9148343.9 i/s
      method_missing:  1729126.6 i/s - 5.29x  slower
```

## rubocop

### Re-Enabled some cops

[Style/BlockDelimiters](https://docs.rubocop.org/en/stable/cops_style/#styleblockdelimiters), [petalmd.rails#5036](https://github.com/petalmd/petalmd.rails/pull/5036)

### rubocop-rspec
On peut regénérer, le `.rubocop_todo.yml` comme sa les cops vont s'appliquer seulement sur les nouveaux fichiers.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTM3OTE1MTQ1MiwtMTkyMDc3NzgyMSw1Mj
I1MzI1MDEsMjEzMjE4NTM2LC05NTAxNTI2ODAsMjc0NTM0NzEx
LDczMDk5ODExNl19
-->