---
layout: default
title: 3 décembre 2020
parent: 2020
grand_parent: Réunion
nav_order: 21
---

# 3 décembre 2020

## Tour de table

## [RSpec mock](../../blog/Rspec mock)

## Rubocop dans les specs

1. Beaucoup de PR pour corrigé le style. 
2. L'activation de dans les specs arrive sous peu [petalmd/petalmd.rails#6349](https://github.com/petalmd/petalmd.rails/pull/6349). 
3. Bientôt, ajouter le plugin rubocop-rpsec

## Nouveau warning `Hash#method_missing`

Je voulais vous proposez d'enlever les warnings qui sont partout et qui ralentisse, pour brutalement lancer une erreur
si l'utilisation du method_missing est employer dans du nouveaux code. 

[Raise on Hash#method_missing in new code](https://github.com/petalmd/petalmd.rails/pull/6375)

Mon plan est d'intégrer cette feature avec la gem Bullet présenté dans un autre chapter.

## Query Elasticsearch

Pensé à ne pas utiliser le DSL de chewy qui est deprecated.

```ruby
AccountsIndex.query({match_phrase_prefix: { first_name: 'Nan'}})
Chewy.client.search(index: AccountsIndex.index_name, body: { query: { match_phrase_prefix: { first_name: 'Nan' } } })

AccountsIndex.filter({term: { first_name: 'nancy'}}).to_a
Chewy.client.search(index: AccountsIndex.index_name, body: { query: { filtered: { query: { match_all: {} }, filter: { term: { first_name: 'nancy' } } } } })
```

