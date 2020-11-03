---
layout: default
title: API
parent: Bon examples
nav_order: 1
grand_parent: Référence
---

# API

## Routing

Grape est extrèmement permissif sur le routing, mais il est possilble de faire des parrèles avec
les convention de Rails. Référence [CRUD, Verbs, and Actions](https://guides.rubyonrails.org/routing.html#crud-verbs-and-actions)

On peut voir le `resources` de Rails pour le `mount` de Grape.

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

**Implémentation:** `app/api/booking_hub/patients_api.rb` 

### Actions custom

Les actions _custom_ devrait être définie avec la méthode HTTP et ne devrais pas nécessité un nouveau
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

**Implémentation:** `app/api/booking_hub/patients_api.rb` 

### Nested resources

Il est possible de faire `mount` dans une resource. La thumb rules est de ne pas avoir plus de un niveau.
Cette recommendation vient de la documentation Rails, [Nested Resources](https://guides.rubyonrails.org/routing.html#nested-resources).
La classe ` mount` devrait donc habituellement contenir un `namespace` par fichier d'API plus un `route_params` maximnum.

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
# GET /api/booking_hub/patients/:id/appointments/appointment_id

mount BookingHub::PatientsApi
```

**Implémentation:** `app/api/booking_hub/patients_api.rb` 

## Params

La définition des paramètres est un des éléments les plus utile de Grape. 
Il y a beaucoup fonctionnalité relier au paramètres, le [README](https://github.com/ruby-grape/grape#parameters)
est le meillieur endroit pour en apprendre plus.

### DRY params et parent namespace

Il est a noté que les paramètres définie en haut d'un group est
définie pour tout les éléments a l'intérieur de ce groupe. Pour éviter
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

**Implémentation:** `app/api/booking_hub/patients_api.rb` 

## Utilisation de service

On utilise des services pour sortir la complexité du endpoint.
C'est utile pour la réutilisation du code et pour la rédaction de
testes unitaires.

Si la complexité est minime,
* 5 lignes de code ou moins
* utilisation de 1 ou 2 classes

l'utilisation d'un service n'est pas nécessaire.

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

## Serializer

Grape va appeler `.to_json` sur l'object retourner à la fin du endpoint. Il faut donc retourner un object qui répond a cette méthode, example
un ActiveRecord ou un Serializer.

L'utilisation d'un serializer est recommender, il permet de selection les attribues a retourner et convertir les
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



