---
layout: default
title: Elasticsearch
parent: Référence
nav_order: 4
---

# Elasticsearch

## Dupliquer dans le nouveau cluster

La migration doit ce faire en 3 étapes.

1. Créer le nouvel index
2. Reset (créer et tout importer) dans le novuel index
3. Modifier tout les endroit où on lit et écrit dans le vieux index pour le nouveau
4. (Bonus) faire le ménage du code pour enlver le vieux index


Voici un exemple concret avec `Schedule::TaskKindsIndex`

### Créer le nouvel index

1. Créer le nouveau fichier sous `app/chewy/new_cluster` avec les mêmes module et nom de classe.

```text
app/chewy/new_cluster/schedule/task_kinds_index.rb
```

2. Copier coller le contenu de la classe d'origine dans le nouveau fichier.
3. Faire les modification suivante:
    * Ajouter le `module NewCluster`
    * Hériter de `< ::NewCluster::Chewy::Index`
    * Si il y a lieu, modifier les `include SomeModule` pour `include ::NewCluster::SomeModule`
4. Changer les `type: :string` => `type: :text` et
   `type: :string, index: :not_analyzed` => `type: :keyword`
   
Le résultat devrait ressembler à ceci:

```ruby
# app/chewy/new_cluster/schedule/task_kinds_index.rb
module NewCluster
  module Schedule
    class TaskKindsIndex < ::NewCluster::Chewy::Index
      define_type ::Schedule::TaskKind do
        field :id, type: :integer
        ...
      end
    end
  end
end
```

5. Ajouter `extend ::NewCluster::SyncNewClusterConcern` dans le vieux index

Une fois ce changement déployé le nouveau index va ce créer automatiquement au besoin. Tout les `import` et `update` dans le vieux
index vont également être fais dans le nouveau (mais pas inversement) pour que tout soit synchronisé.

**Prendre note** que les opérations sur la class index n'affecte pas le nouvel index. Exemple, `Schedule::TaskKindsIndex.import` 
ne va pas tout réimporter dans `NewCluster::Schedule::TaskKindsIndex`. Seul les opérations sur la class type va s'effectuer sur les 2.
Exemple, `Schedule::TaskKindsIndex::TaskKind.import` va tout réimporter dans le vieux comme dans le nouveau `NewCluster::Schedule::TaskKindsIndex::TaskKind`.

## Reset

Demander au Chapter Backend de lancer un reset du nouveau index. Exemple de commande pour test sur stg.

```bash
bundle exec rake chewy:sidekiq:reset['new_cluster/schedule/task_kinds']
```

## Modification lecture, écriture

1. Faire le tour de l'app et test pour changer la class de l'index pour la nouvelle `Schedule::TaskKindsIndex` => '::NewCluster::Schedule::TaskKindsIndex'.
2. Faire le tour des `update_index` pour utiliser le nouveau index `schedule/task_kinds#task_kind` => `new_cluster/schedule/task_kinds#task_kind`
3. Modifier les query pour supporter le nouveau DSL et vérifier la compatibilité avec ES 7. Voir la section [new_query_dsl](#new_query_dsl) pour plus de détails.


Une fois déployé, le vieux index ne sera en théorie plus mis à jour. 

## New query DSL

https://github.com/toptal/chewy/tree/v5.2.0#legacy-dsl-incompatibilities

`only(:id)` => `source(:id)`

## Mapping

### id

`field :id, type: :keyword`

Le type `keyword` est le type le plu performant. Puisqu'on pratiquement jamais de query _range_, _greater than_ mais juste des filters
`term` le plus efficace c'est de les mapper en `keyword`.

> **Mapping numeric identifiers**
> 
> Not all numeric data should be mapped as a numeric field data type. Elasticsearch optimizes numeric fields, such as integer or long,
> for range queries. However, keyword fields are better for term and other term-level queries.
> 
> Identifiers, such as an ISBN or a product ID, are rarely used in range queries. However, they are often retrieved using term-level queries.
> 
> Consider mapping a numeric identifier as a keyword if:
> * You don’t plan to search for the identifier data using range queries.
> * Fast retrieval is important. term query searches on keyword fields are often faster than term searches on numeric fields.
> 
> If you’re unsure which to use, you can use a multi-field to map the data as both a keyword and a numeric data type.
> 
> Source: [Keyword type family](https://www.elastic.co/guide/en/elasticsearch/reference/7.11/keyword.html)

### Searchable

Faire du `multi_match` sur plusieurs champs ou un gros `should` peut ralentir la recherche. Une façon de chercher sur 
1 seul champ a la fois est de contatenner tout les champs avec lequels on cherche habituellement (`first_name`, `last_name`)
dans 1 seul champ `searchable`. C'est facilement réalisable durant le mapping avec `copy_to`.

Cela permetterait de définire le pattern suivant:

Si on définie un champs de comment on peut chercher un object, example: Account, Patient où c'est 2 index aurait un champ
`searchable` on pourrait très très facilement créer une recherche multi index avec `accounts,patients` qui utiliserait
le champ `searchable`. Un exemple réelle est la console, la même bar de recherche va regardé `task_kinds#event_name/abbreviation`,
`console_layout_rows#label`, `groups#acronym/code/name`.

**Source**

* https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-search-speed.html#search-as-few-fields-as-possible
* https://www.elastic.co/guide/en/elasticsearch/reference/current/copy-to.html
* https://github.com/forem/forem/blob/d2d9984f28b1d0662f2a858b325a0e6b7a27a24c/config/elasticsearch/mappings/users.json#L57

### Multiple fields

Comme règle je proposerais d'avoir des root field en `keyword` avec des sub-fields pour chercher. Voici un exemple:

Les licences: "QC-1234". On pourrait avoir un `field` avec `territory` et un autre `number` juste pour chercher.

```ruby
PUT license
{
  "settings": {
    "analysis": {
      "analyzer": {
        "standard_digits": {
          "tokenizer": "digits"
        }
      },
      "tokenizer": {
        "digits": {
          "type": "pattern",
          "pattern": ["(\\D+)"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "license": {
        "type": "keyword",
        "fields": {
          "territory": {
            "type": "text",
            "analyzer": "simple"
          },
          "number": {
            "type": "text",
            "analyzer": "standard_digits"
          }
        }
      }
    }
  }
}
```

