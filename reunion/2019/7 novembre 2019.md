---
layout: default
title: 7 novembre 2019
parent: 2019
grand_parent: Réunion
nav_order: 3
---

# 7 novembre 2019

- Tour de table

---

- Roadmap

---

## [Epic Rails 5]([https://petalmd.atlassian.net/browse/RAILS-328](https://petalmd.atlassian.net/browse/RAILS-328))
- 39 gems a updated (liste dans l'epic)
	- Supprimer le _Gemfile.lock_ et faire `bundle install`, conséquence met tout à jour en respectant le _Gemfile_
	- Une gem par sprint par dev?
- _protected_attributes_ ➡️ _strong_parameters_

## Crash course updater une gem

- Lister les gem _outdated_
	- `bundle outdated  --only-explicit --strict --group=test`
```
Outdated gems included in the bundle:
===== Group test =====
  * factory_bot_rails (newest 5.1.1, installed 4.11.1)
  * faker (newest 2.2.1, installed 1.9.1)
  * json_matchers (newest 0.11.1, installed 0.7.0)
  * mock_redis (newest 0.22.0, installed 0.19.0)
  * nokogiri (newest 1.10.5, installed 1.10.4)
  * rspec-rails (newest 3.9.0, installed 3.8.0)
  * rubocop-rspec (newest 1.36.0, installed 1.30.0)
  * simplecov (newest 0.17.1, installed 0.15.1)
  * timecop (newest 0.9.1, installed 0.8.1)
  * webmock (newest 3.7.6, installed 3.5.1)
```

### Example `factory_bot_rails`
- Trouver la gem sur RubyGems [factory_bot_rails](https://rubygems.org/gems/factory_bot_rails)
1. Regarder les _runtime dependencies_
2. Pour _railties_ `bundle info railties`
```
 * railties (4.2.11.1) ✅
        Summary: Tools for creating, working with, and running Rails applications.
        Homepage: http://www.rubyonrails.org
        Path: /Users/bhacaz/.rbenv/versions/2.4.9/lib/ruby/gems/2.4.0/gems/railties-4.2.11.1
```
3. Pour _factory_bot_ `bundle info factory_bot`
```
  * factory_bot (4.11.1) ❌
        Summary: factory_bot provides a framework and DSL for defining and using model instance factories.
        Homepage: https://github.com/thoughtbot/factory_bot
        Path: /Users/bhacaz/.rbenv/versions/2.4.9/lib/ruby/gems/2.4.0/gems/factory_bot-4.11.1
```
- La gem _factory_bot_ va aussi devoir être updater.
- Trouver le `CHANGELOG` sur Github de _factory_bot_ (peut aussi être dans [releases](https://github.com/thoughtbot/factory_bot/releases))
- Faire l'update: `bundle update <<gem>> --conservative`
	- > Use bundle install conservative update behavior and do not allow shared dependencies to be updated. [Overlapping Dependencies](https://bundler.io/man/bundle-update.1.html#OVERLAPPING-DEPENDENCIES)
- Faire les changements nécessaire pour le bon fonctionnement de l'app
- Créer une PR

## Source d'information continue 

- [https://rubyweekly.com/](https://rubyweekly.com/)
- [http://www.rubyflow.com/](http://www.rubyflow.com/)
- [http://rubyland.news/](http://rubyland.news/)
- [https://weblog.rubyonrails.org/news/](https://weblog.rubyonrails.org/news/)
- dev.to - [https://dev.to/t/ruby](https://dev.to/t/ruby) - [https://dev.to/t/rails](https://dev.to/t/rails)
- Regrouper avec [Feedly](https://feedly.com/i/collection/content/user/81719656-683a-46b1-9cdb-4a0ff2bc4426/category/be5929a3-6063-405b-88a8-cb972c4e7132)


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTcwMjcwNDc4OSwtNzE4NDExMjQ5LDYwOT
M0ODA1MSwtNzk1MDY2MjUxLDkzNDYxNDczMiwxNzcxNzQ2OTAs
LTE3OTQ3NDQwNjAsLTIxMzMzMDQ4MTYsLTkzMTQ0NTQyNCwtOT
A3MTA0MzU4LC0xNDM4MTI3NTAxXX0=
-->
