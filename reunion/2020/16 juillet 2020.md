---
layout: default
title: 16 juillet 2020
parent: 2020
grand_parent: Réunion
nav_order: 13
---

# 16 juillet 2020

## Tour de table

## Rails 5.1 🎊

Update de Rails 5.1 le 7 juillet 2020 [petalmd/petalmd.rails#5715](https://github.com/petalmd/petalmd.rails/pull/5715)

## Nouveautés Rails 5.1 ([Release notes](https://guides.rubyonrails.org/5_1_release_notes.html))

### Webpack

Possibiliter d'utiliser _webpack_ au lieu de l'asset pipeline `rails new app --webpack`. 

_C'est pas dans les plans de faire le switch._

### `Date#all_day`

Très pratique dans un where. Example, les messages qui ont été créé aujourd'hui:

```ruby
>> Messaging::Message.where(created_at: Date.current.all_day).count
   (8.9ms)  SELECT COUNT(*) FROM `mess__messages` WHERE (`mess__messages`.`created_at` BETWEEN '2020-07-15 00:00:00' AND '2020-07-15 23:59:59')
```


## ~~Overcommit~~ et Git hooks

**Suggestion:** [petalmd/petalmd.rails#5742](https://github.com/petalmd/petalmd.rails/pull/5742)

1. Enlever la gem Overcommit
2. Ajouter des githooks dans le repo (l'installation reste facultative)
3. Avez vous des idées de hook/script

## `Arel.sql()` (Rails 5.2 preview)

[Disallow raw SQL in dangerous AR methods (rails/rails#27947)](https://github.com/rails/rails/pull/27947/files)

Pour les méthodes `pluck` (`pluck_h`) et `order` il y a maintenat une sécurité de plus pour l'injection de SQL.
Si les paramètres ne match pas ce [pattern](https://github.com/rails/rails/blob/a1ee43d2170dd6adf5a9f390df2b1dde45018a48/activerecord/lib/active_record/attribute_methods.rb#L170-L180
) sa lance une erreur:

```
Regexp whitelist. Matches the following:
  "#{table_name}.#{column_name}"
  "#{table_name}.#{column_name} #{direction}"
  "#{column_name}"
  "#{column_name} #{direction}"

* direction (asc, desc)
```

La solution est de simplement mettre la string dans un `Arel.sql()` par example.

```ruby
relation.pluck(Arel.sql('sche__memberships.group_id AS group_id'))
````


