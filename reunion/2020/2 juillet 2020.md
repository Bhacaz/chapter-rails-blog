---
layout: default
title: 2 juillet 2020
parent: 2020
grand_parent: Réunion
nav_order: 12
---

# 2 juillet 2020

## Tour de table

## [Enum prefix](https://api.rubyonrails.org/v5.0.7.2/classes/ActiveRecord/Enum.html)

Rails est reconnue pour faire de la magie. Il crée beacoup de méthodes avec du metaprogramming. Avant les versions, Rails nous donne
de plus en plus d'avertissement sur ce qui se passe pour ne pas créer de conflit.
Vous avez surment remarqué les warnings depuis Rails 5.0: 

```
Creating scope :included. Overwriting existing method Simple::ResourceSelectorResource.included.
```

Ici le `included` vient de [Module#included](https://devdocs.io/ruby~2.6/module#method-i-included)

Quand on définie un [*enum*](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/enum.rb#L160) on crée des scopes example: 

```ruby
module Patient
  class Subscription < ApplicationRecord
    enum period_opening_notification_kind: { nothing: 0, each: 1, next: 2, specific_date: 3 }
  end
end

Patient::Subscription.nothing.count 
# => (2.4ms)  SELECT COUNT(*) FROM `pati__subscriptions` WHERE `pati__subscriptions`.`period_opening_notification_kind` = 0
```

Avant de crée le scope Rails fait [plusieurs vérification](https://github.com/rails/rails/blob/4fe1452534540851461e99b3d0fecaf27931ab93/activerecord/lib/active_record/enum.rb#L255),
 si l'object implément déjà la méthodes. 

En Rails 5.2, ce n'est plus juste des warnings... C'est des erreurs, 
du moins quand la méthodes qu'on va overwriter est définie dans *ActiveRecord::Relation*

Si on revient a l'example: 

```ruby
# enum period_opening_notification_kind: { nothing: 0, each: 1, next: 2, specific_date: 3 }
#                                                      ^^^^
>> Patient::Subscription.all.each(&:touch)
```

Devrait faire un `touch` sur tout les records...

```ruby
>> Patient::Subscription.all.count
# => 4

>> Patient::Subscription.all.each.count
# => 1
```

### Retour au préfix

Spécifier un préfix permet de changer le nom des scopes qui seont créer.

```ruby
class ContactMethod < ApplicationRecord
    enum visibility: { hide: 0, display: 1, registry_only: 2 }, _prefix: true
end

>> ContactMethod.display
# devient
>> ContactMethod.visibility_display
```

Ce qui je crois fait beaucoup de sens et reste très lisible sinon plus. Et sa réduit les change d'overwriter des méthodes.