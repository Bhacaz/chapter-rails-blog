---
layout: default
title: 30 janvier 2020
parent: 2020
grand_parent: Réunion
nav_order: 1
---

# 30 janvier 2020

## Tour de table

---

## Tester les assets

[Avril 2017](https://github.com/petalmd/petalmd.rails/pull/1073), Certaine gem des assets on été updaté (coffee-rails, sprockets, compass-rails, ...) pour passer en Rails 5.

Une heure plus tard **revert!**, le schedule n'est plus accessible.

Sa fonctionnait en development et staging

### Comment bien testé

Utilisé une config qui **resemble a la prod**.

1. Modifier `config/environments/development.rb`

	```ruby
	config.assets.debug = false
	config.assets.compile = false
	config.assets.digest = true
	```

2. Enlever les gems `quiet_assets` et `sprockets-derailleur` du le Gemfile (elles sont seulement utilisé en development et test)
3. `bundle install`
4. Effacer les assets déjà compilé `bin/rake assets:clobber`
5. Compiler les assets `bin/rake assets:precompile`
6. Redémarrer le serveur.
7. 🤞

---

## [Autoloading and Reloading Constants](https://guides.rubyonrails.org/autoloading_and_reloading_constants_classic_mode.html)

[(Rails 6: Zeitwerk Mode)](https://guides.rubyonrails.org/autoloading_and_reloading_constants.html)

### Ruby pure

```ruby
# account.rb
class Account; end

# membership.rb
require 'account'
class Membership; end
```

### Avec Rails
[Algo](https://guides.rubyonrails.org/autoloading_and_reloading_constants_classic_mode.html#resolution-algorithm-for-relative-constants)

Overwrite le `constant missing` du Kernel

```ruby
>> Abc
NameError: uninitialized constant Abc
        from (irb):1
        from /Users/bhacaz/.rbenv/versions/2.4.9/bin/irb:11:in `<main>'
```

### Resolution du path

`String#underscore` (ActiveSupport)

```ruby
>> 'Messaging::ConverserMessage'.underscore
# => "messaging/converser_message"
```

```ruby
>> 'Connection::OpenIDMicrosft'.underscore
# => "connection/open_id_microsft"
```

---

