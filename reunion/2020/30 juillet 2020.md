---
layout: default
title: 30 juillet 2020
parent: 2020
grand_parent: Réunion
nav_order: 14
---

# 30 juillet 2020

## Tour de table

## `Enumerable#pluck` a la rescousse

J'ai souvent parler du `Hash#method_missing`

```ruby
array = [{ id: 1 }, { id: 2 }]

# bad
array.map(&:id)

# good
array.map { |h| h[:id] }
```

Depuis Rails 5 ActiveSupport ajoute `Enumerable#pluck`

* [rails/rails#20350](https://github.com/rails/rails/pull/20350)
* [rails/rails#20362](https://github.com/rails/rails/pull/20362)


```ruby
# File activesupport/lib/active_support/core_ext/enumerable.rb, line 107

def pluck(*keys)
  if keys.many?
    map { |element| keys.map { |key| element[key] } }
  else
    map { |element| element[keys.first] }
  end
end
```

On a maintenant un racourcie pour corriger les `Hash#method_missing`

```ruby
# good
array.pluck(:id)
```

On peut l'utiliser de la même façon que avec un `ActiveRecord::Relation`

```ruby
[{ name: "David" }, { name: "Rafael" }, { name: "Aaron" }].pluck(:name)
# => ["David", "Rafael", "Aaron"]

[{ id: 1, name: "David" }, { id: 2, name: "Rafael" }].pluck(:id, :name)
# => [[1, "David"], [2, "Rafael"]]
```

## `pluck` VS `select`

Le `select` peut être très pratique et efficace.

```ruby
period = Schedule::Period.find(456); nil

# 1
accounts = period.resources.map(&:account)
#   Schedule::Resource Load (3.6ms)  SELECT `sche__resources`.* FROM `sche__resources` WHERE `sche__resources`.`period_id` = 456 ORDER BY id ASC
#    Account Load (4.4ms)  SELECT  `accounts`.* FROM `accounts` WHERE `accounts`.`id` = 146 LIMIT 1
#    Account Load (3.3ms)  SELECT  `accounts`.* FROM `accounts` WHERE `accounts`.`id` = 149 LIMIT 1
#    Account Load (2.9ms)  SELECT  `accounts`.* FROM `accounts` WHERE `accounts`.`id` = 150 LIMIT 1
#    Account Load (3.0ms)  SELECT  `accounts`.* FROM `accounts` WHERE `accounts`.`id` = 145 LIMIT 1
#    Account Load (3.0ms)  SELECT  `accounts`.* FROM `accounts` WHERE `accounts`.`id` = 144 LIMIT 1
#    Account Load (3.1ms)  SELECT  `accounts`.* FROM `accounts` WHERE `accounts`.`id` = 148 LIMIT 1

# 2
Account.joins(memberships: { group: { periods: :resources } }).merge(Schedule::Resource.where(period_id: period.id))
#   Account Load (13.2ms)  SELECT `accounts`.* FROM `accounts` INNER JOIN `memberships` ON `memberships`.`account_id` = `accounts`.`id` INNER JOIN `groups` ON `groups`.`id` = `memberships`.`group_id` INNER JOIN `sche__periods` ON `sche__periods`.`group_id` = `groups`.`id` INNER JOIN `sche__resources` ON `sche__resources`.`period_id` = `sche__periods`.`id` WHERE `sche__resources`.`period_id` = 456

# 3
Account.where(id: period.resources.pluck(:account_id))
#    (3.0ms)  SELECT `sche__resources`.`account_id` FROM `sche__resources` WHERE `sche__resources`.`period_id` = 456 ORDER BY id ASC
#    Account Load (2.4ms)  SELECT `accounts`.* FROM `accounts` WHERE `accounts`.`id` IN (146, 149, 150, 145, 144, 148)

# Finalement
Account.where(id: period.resources.select(:account_id))
#   Account Load (3.4ms)  SELECT `accounts`.* FROM `accounts` WHERE `accounts`.`id` IN (SELECT `sche__resources`.`account_id` FROM `sche__resources` WHERE `sche__resources`.`period_id` = 456 ORDER BY id ASC)
```

* Pas de N+1
* Moins long a écrire et query MySQL plus simple et plus rapide
* Moins de query BD (forcer par le `pluck`)

### Une cop pour les trouvés

**⚠️ Attention au faux positif**

[`Rails/PluckInWhere`](https://github.com/rubocop-hq/rubocop-rails/issues/246)

## VCR 📼

Gem que j'ai entendu parlé qui pourrait être intéressant dans plusieurs context, en autre dans booking.
Du moins il est intéressant de savoir que sa existe.

Nous avons déjà `webmock` pour éviter les call http sortant durant les tests. Sinon sa
serait trop long et on pourrait tomber sur des erreurs de rate limit.

La gem [vcr](https://github.com/vcr/vcr) permet de mocker facilement ces appel
en les exécutant pour de vrai un fois et en enregistrant les paramètres du call dans un fichier 
 `.yml` que la gem appel cassette. Donc la prochaine fois qu'on va vouloir éxécuter cette requête,
 on peut simplement rejouer la cassette.

### Example

Je veux récupérer l'url de l'avatar d'un user avec l'api Github.

```ruby
def getGithubProfilePicture(profile)
  response = HTTParty.get("https://api.github.com/users/#{profile}")
  JSON.parse(response)['avatar_url']
end
```

Aujourd'hui on pourait faire quelque chose du genre:

```ruby
let(:profile) { 'petalmd' }
it 'return the avatar_url' do
  allow(HTTParty).receive(:get).with("https://api.github.com/users/#{profile}") { {'avatar_url' => 'example.com'}.to_json }
  expect(getGithubProfilePicture(profile)).to eq('example.com')
end
```

Si dans 1 mois Github changait `avatar_url` pour `avatar_uri` notre CI le trouverais pas.
Ou si un DME changait la signature de leur API même chose.

### Example complet avec VCR

```ruby
def getGithubProfilePicture(profile)
  response = HTTParty.get("https://api.github.com/users/#{profile}")
  JSON.parse(response)['avatar_url']
end

require 'httparty'
require "vcr"

VCR.configure do |c|
  c.cassette_library_dir = "spec/vcr"
  c.hook_into :webmock
  c.ignore_localhost = true
end

describe 'Test VCR' do
  let(:user_response) do
    VCR.use_cassette("github/user", re_record_interval: 1.week.to_i) { getGithubProfilePicture('petalmd') }
  end

  it { expect(user_response).to match(URI::regexp) }
end
```

L'intéret que je vois c'est de mêtre l'options `re_record_interval` par example 
1 fois semaine pour valider que la signature des appels d'API est la même et qu'on
l'identifie dans le CI.

**Est-ce que sa vous inspire des idées?**

## [Introduction a Rack](../../blog/Introduction a Rack)
