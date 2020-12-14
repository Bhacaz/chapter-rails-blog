---
layout: default
title: Service
parent: Bon examples
nav_order: 2
grand_parent: Référence
---

# Service

On peut répertorier 2 types de service,

1. D'API
2. Générique

## Service d'API

Les services d'API servent a extraire la logique d'un appel HTTP. La structure peut ressemble au un controlleur classique
avec une méthode par endpoint/action.

L'initializer peut être pratique pour passer l'`authentificated_account` ou bien un `route_params` pour être utiliser
comme variable d'instance.

Pour bien séparer les params du call d'API et le service, `params` ne devrait être passer directement, mais être séparer
en argument lorsque passer à la méthode.

On peut rajouter des méthodes publique dans la classe qui peuvent service de _helper methods_.

```ruby
# app/api/v2/group/seniority/seniority_api.rb

class SeniorityApi < Grape::API

  after_validation do
    @service = SeniorityService.new(params[:group_id])
  end

  resource :seniority do
    get do
      @service.get_seniorities(params[:membership_ids])
    end

    post :reorder do
      @service.reorder_seniority(params[:membership_ids], params[:query])
    end

    get :unclassified_exists do
      @service.unclassified_exists?
    end
  end
end

# app/services/groups/memberships/seniority_service.rb

class SeniorityService
    def initialize(group_id)
      ...
    end
    
    def reorder_seniority(membership_ids)
      ...
    end
    
    def get_seniorities(membership_ids, query)
      ...
    end
    
    def unclassified_exists?
      ...
    end

    # Helper methods

    def some_method; end
    
    private
    
    def search_accounts(query)
      ...
    end
end
```

**Implémentation:** [app/services/groups/memberships/seniority_service.rb](https://github.com/petalmd/petalmd.rails/blob/master/app/services/groups/memberships/seniority_service.rb)

## Service générique (_PROPOSITION_)

[Ruby on Rails pattern: Service Objects](https://dev.to/joker666/ruby-on-rails-pattern-service-objects-b19)
[Rails Service Objects: A Comprehensive Guide](https://www.toptal.com/ruby-on-rails/rails-service-objects-tutorial)

Ce type de service existe pour être réutiliser très facilement et répondre à un besoin très spécifique.
Pour son implémentation il est recommenté de suivre
le concept de classe [SOLID](https://en.wikipedia.org/wiki/SOLID), bien que cela peut ne pas s'appliquer à tout
les besoins. [`Common::ProductService`](https://github.com/petalmd/petalmd.rails/blob/master/app/services/common/product_service.rb) 
est un bon exemple de service non _callable_ avec plusieurs méthode mais qui répond a un besoi très précis.

Un des patterns qui peu être employer est de créer une classe dite `callable`.

```ruby
class FileCreatorService
  def self.call(data, timestamp_start_time, group_id)
    ...
  end
end
```

Si la complexité requière une instance de la classe voici une suggestion de pattern.

```ruby
class ApplicationService
  def self.call(*args, &block)
    new(*args, &block).call
  end
end

class FileCreatorService < ApplicationService
  def new(data, timestamp_start_time, group_id)
    @data = data
    @timestamp_start_time = timestamp_start_time
    @group_id = group_id
  end

  def call
    ...
  end

  private

  def other_method
    ...
  end
end

FileCreatorService.call(data, timestamp_start_time, group_id)
```

**Implémentation:** : [`app/services/hospital/export/file_creator.rb`](https://github.com/petalmd/petalmd.rails/blob/master/app/services/hospital/export/file_creator.rb)
