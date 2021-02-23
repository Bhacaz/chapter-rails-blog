---
layout: default
title: API
parent: Bon exemples
nav_order: 1
grand_parent: Référence
---

# API

## Routing

Grape est extrèmement permissif sur le routing, mais il est possilble de faire des parrèles avec
les conventions de Rails, [CRUD, Verbs, and Actions](https://guides.rubyonrails.org/routing.html#crud-verbs-and-actions).

On peut voir le `resources` de Rails comme le `mount` de Grape.

### Basic CRUD

**Exemple**

```ruby
module BookingHub
  class PatientsApi < Grape::API
    resource :patients do # Commencer par le namespace
      get { }
      post { }

      route_params :id do
        get { }
        patch { }
        put { }
        delete { }
      end
    end
  end
end

# GET /api/booking_hub/patients
# POST /api/booking_hub/patients
# GET /api/booking_hub/patients/:id
# PATCH /api/booking_hub/patients/:id
# PUT /api/booking_hub/patients/:id
# DELETE /api/booking_hub/patients/:id

mount BookingHub::PatientsApi
```

**Implémentation:** [`app/api/booking_hub/patients_api.rb`](https://github.com/petalmd/petalmd.rails/blob/master/app/api/booking_hub/patients_api.rb) 

### Actions custom

Les actions _custom_ devraient être définies avec la méthode HTTP et ne devraient pas nécessités un nouveau
namespace.

**Exemple**

```ruby
module BookingHub
  class PatientsApi < Grape::API
    resource :patients do # Commencer par le namespace
      route_params :id do
        get(:contact_informations) { } 
        put(:contact_informations) { } 
      end
    end
  end
end

# GET /api/booking_hub/patients/:id/contact_informations
# PUT /api/booking_hub/patients/:id/contact_informations

mount BookingHub::PatientsApi
```

**Implémentation:** [`app/api/booking_hub/patients_api.rb`](https://github.com/petalmd/petalmd.rails/blob/master/app/api/booking_hub/patients_api.rb) 

### Nested resources

Il est possible de faire un `mount` dans une resource. La _thumb rules_ est de ne pas avoir plus de deux niveaux de resources.
Cette recommendation vient de la documentation Rails, [Nested Resources](https://guides.rubyonrails.org/routing.html#nested-resources).
La classe qui a été `mount` devrait donc habituellement contenir qu'un seul `resource` plus un `route_params` maximnum.

**Exemple**

```ruby
module BookingHub
  class PatientsApi < Grape::API
    resource :patients do # Commencer par le namespace
      route_params :id do
        get { }
        patch { }
        put { }
        delete { }

        mount BookingHub::Patients::AppointmentsApi
      end
    end
  end
end

module BookingHub
  module Patients
    class AppointmentsApi < Grape::API
      resource :appointments do # Commencer par le namespace
        get { }

        route_param :appointment_id do
          get { }
        end
      end
    end
  end
end

# GET /api/booking_hub/patients/:id
# PATCH /api/booking_hub/patients/:id
# PUT /api/booking_hub/patients/:id
# DELETE /api/booking_hub/patients/:id
# GET /api/booking_hub/patients/:id/appointments
# GET /api/booking_hub/patients/:id/appointments/:appointment_id

mount BookingHub::PatientsApi
```

**Implémentation:** [`app/api/booking_hub/patients_api.rb`](https://github.com/petalmd/petalmd.rails/blob/master/app/api/booking_hub/patients_api.rb) 

## Params

La définition des paramètres est un des éléments les plus utile de Grape. 
Il y a beaucoup fonctionnalité relier aux paramètres, le [README](https://github.com/ruby-grape/grape#parameters)
est le meillieur endroit pour en apprendre plus.

### DRY params et parent namespace

Il est a noté que les paramètres définie en haut d'un groupe est
définie pour tout les éléments a l'intérieur de celui-ci. Pour éviter
la redondance et pour des quesitons de performance il est préférable de
définir une fois le param. Pour en savoir plus consulter le [README](https://github.com/ruby-grape/grape#include-parent-namespaces)
de Grape.

**Exemple**

```ruby
module BookingHub
  class PatientsApi < Grape::API
    resource :patients do

      params do
        requires :id, type: Integer, desc: 'Patient id'
      end

      route_params :id do
        # params[:id] est require pour tout les endpoints
        get { }
        patch { }
        put { }
        delete { }
      end
    end
  end
end
```

**Implémentation:** [`app/api/booking_hub/patients_api.rb`](https://github.com/petalmd/petalmd.rails/blob/master/app/api/booking_hub/patients_api.rb) 

## Authorization

La vérification de permissions et d'accès devrait être fais a même la classe d'API. Ces vérification peut généralement être
faitent dans un callback et s'appliquer a tout les endpoints.

```ruby
class ConversationsApi < Grape::API
  # Validation avant le namepspace pour s'appliquer sur tout celui-ci
  after_validation do
    raise ApiErrors::ForbiddenError.new('Forbidden Error', track: false) unless authenticated_account.fetch_groups.any? do |group|
      Common::ProductService.is_available?('PRODUCT_GROUP_MESSAGING_CHAT', group)
    end
  end

  resources :messaging do
    get { }
    post { }
  end
end

```

## Utilisation de service

On utilise des services pour sortir la complexité du endpoint.
C'est utile pour la réutilisation du code et pour la rédaction de
testes unitaires.

L'utilisation d'un service n'est pas essentiel, si la complexité est minime:
* 5 lignes de code ou moins
* utilisation de 1 ou 2 classes

**Exemple**

```ruby
module BookingHub
  class PatientsApi < Grape::API
    resource :patients do
      get(:find) { ::BookingHub::PatientService.find(params) }
      route_params :id do
        get { Patient::Patient.find(params[:id]) }
      end
    end
  end
end
```

**Implémentation:** [`app/api/booking_hub/patients_api.rb`](https://github.com/petalmd/petalmd.rails/blob/master/app/api/booking_hub/patients_api.rb) 

## Serializer

Grape va appeler `.to_json` sur l'object retourner à la fin du endpoint. 
Il faut donc retourner un object qui répond a cette méthode, example
un ActiveRecord ou un Serializer.

L'utilisation d'un serializer est recommender, il permet de selectionner les attribues à retourner et convertir les
clé pour le frontend.

**Exemple**

```ruby
module BookingHub
  class PatientsApi < Grape::API
    resource :patients do
      route_params :id do
        get do
          patient = Patient::Patient.find(params[:id])
          PatientSerializer.new(patient)  
        end
      end
    end
  end
end
```

**Implémentation:** [`app/api/booking_hub/patients_api.rb`](https://github.com/petalmd/petalmd.rails/blob/master/app/api/booking_hub/patients_api.rb) 

## Tester les API

Un test d'API devrait avoir c'est 2 sections:

1. Cas classique de succès (minimum de paramètre pour avoir un _success_), qui regarde également
    * Que le code fonctionne bien ou qu'il appel (mock) bien un service
    * L'utilisation et/ou le contenue de la réponse. Pour tester un minumum le serializer.
2. _Success_ avec des éléments optionel
    * Comme des paramètres optional (query, per_page, page)
2. _Bad request_
    * Sur les params: validation, présence. Il n'est pas nécessaire de tester tout les cas possibles. 
    Seulement les plus spécifiques, relatif à l'API testé.
    * Permission, produit, etc

Il est recommender d'utiliser des mocks pour plusieurs éléments comme:
* Les middlewares
* La disponibilité des produits
* Les services

Cela permet d'accélérer l'exécution des tests, de les simplifiers et de ne pas tester 2x la même chose.

<details>

<summary>Exemple</summary>

{% highlight ruby %}
describe V2::Messaging::ConversationsApi do
  let(:authenticated_account) { FactoryBot.build_stubbed :confirmed_account }

  before :each do
    # Mock authentication
    allow(Common::SecurityService).to receive(:authenticate!)
    allow(Common::SecurityService).to receive(:current_user) { authenticated_account }
  end

  describe 'POST /api/v2/conversations/:id' do
    subject do
      patch path, params: params
      response
    end

    let(:path) { "/api/v2/conversations/#{conversation.id}" }
    let(:params) do
      {
        message_text: message_text,
        message_type: message_type
      }
    end

    let(:conversation) { FactoryBot.create(:messaging_conversation, account: account) }
    let(:message_text) { 'test message' }
    let(:message_type) { 'urgent' }
    let(:message) { FactoryBot.build_stubbed :messaging_message, conversation: conversation }

    context 'successful call' do
      it 'is successful' do
        expect_any_instance_of(Messaging::ConversationsService).to(
          receive(:create_message).with(conversation.id, message_text, message_type, [], [], nil) { message }
        )
        expect(::Messaging::MessageSimpleSerializer).to receive(:new).and_call_original
        is_expected.to be_successful
      end

      context 'with handsake' do
        let(:handshake) { 'this_is_it' }
        let(:params) do
          {
            message_text: message_text,
            message_type: message_type,
            handshake: handshake
          }
        end

        it 'is successful' do
          expect_any_instance_of(Messaging::ConversationsService).to(
            receive(:create_message).with(conversation.id, message_text, message_type, [], [], handshake)
          )
          is_expected.to be_successful
        end
      end
    end

    context 'when the product is not available' do
      it 'is forbidden' do
        allow(Common::ProductService).to receive(:is_available?).with('PRODUCT_GROUP_MESSAGING_CHAT', group) { false }
        expect(subject).to be_forbidden
      end
    end
  end
end

{% endhighlight %}

</details>

**Implémentation**: [`spec/api/v2/messaging/conversations_api_spec.rb`](https://github.com/petalmd/petalmd.rails/blob/master/spec/api/v2/messaging/conversations_api_spec.rb)

