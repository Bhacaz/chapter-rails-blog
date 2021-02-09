---
layout: default
title: Introduction ElasticSearch
parent: Blog
nav_order: 9
---

# Introduction ElasticSearch

## C'est quoi?

> You know, for search (and analysis)
>  
> [What is Elasticsearch?](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/elasticsearch-intro.html#elasticsearch-intro)

C'est une base de données qui permet de préanalyzer des données et ainsi exécuter des query très rapidement.
Les documents sont enregistrer en JSON et on parle a elasticsearch avec une REST API classique.

## Installer ElasticSearch

Le plus simple c'est avec Docker, voici un [_docker-compose.yml_](https://gist.github.com/Bhacaz/eb54c1cf3dd42adc842c96b225e0fcff)
pour commencer facilement.

Il y a 2 images:

1. ElasticSearch:  Base de donnée et le backend (REST API). ([http://localhost:9200](http://localhost:9200/))
2. Kibana: Une interface pour parler à ElasticSearch ([http://localhost:5601](http://localhost:5601/app/home), [console](http://localhost:5601/app/dev_tools#/console))

## Concept de base

* Indice: Peuvent être vu comme un **model**
* Documents: Peuvent être vu comme des **records**

Note: Les types sont quelques qui existe dans notre app/version de ES, mais ça n'existe plus.

Avant:

```bash
GET books/novel/1
GET books/db/1
GET books/kid/1
```

Aujourd'hui tout est un `_doc`. On peut mettre un champ `type` ou bien créer plusieurs index et chercher dans plusieurs index à la fois.

```bash
GET books/_doc/1
```

## Essayer ElasticSearch

ElasticSearch est un bac à sable, on peut litéralement directement pousser 
du JSON pour créer des à la volé des indices et des documents. Il utilise une [RESTful API](https://www.elastic.co/guide/en/elasticsearch/reference/current/rest-apis.html) très classique.

Si on pousse du JSON:

```bash
POST test/_doc/1
{
  "text": "hello world"
}
```

C'est optional de spécifier l'id. Mais plus simple pour le fetcher plus tard. Sinon Elasticsearch va en générer un. 

Pour avoir l'info sur le nouvelle indices nouvellement créer:

```bash
GET test
```

Récupérer le document qu'on vient de créer:

```bash
GET test/_doc/1
```

Modifier le document 1:

```bash
PUT test/_doc/1
{
  "text": "hello world2"
}
```

Recherche dans l'indice

```bash
GET test/_search
{
  "query": {
    "match": {
      "text": "hello"
    }
  }
}
```

Supprimer le document

```bash
DELETE test/_doc/1
```

## [Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html)

Pour définir un indice. Prédéfnir les field, leur type, analyzer etc.

```bash
PUT books
{
    "mappings": {
      "properties": {
        "title": {
          "type": 'text',
          "fields": {
            "raw": {
              "type": 'keyword'
            }
          }
        },
        "author": {
          "properties": {
            "name": { "type": 'text' }
          }
        },
        "synopsis": { "type": 'text' },
        "genres": { "type": 'keyword' }
      }
    }
}
```

Plusieurs chose a noté ici:

1. `title` va être composé de 2 field, `title` et `title.raw` qui seront analyzer de 2 façons différentes.
   On pourra donc chercher sur un des 2 fields ou les 2 dépendament des besoins.
2. `author` est une nested, il contient un object.
3. `genres` va au finial contenir un array. Il y a pas de type `array`, ElasticSearch ne fait pas de différence, il va simplement faire les mêmes opérations
sur tout les éléments de l'array.
4. `text`et `keyword` sont 2 types qui représente du texte (String), la différence c'est qu'il un analyzer par défaut qui leur sont associées.

Voici quelque types qui peuvent être intéressant de savoir qu'il existe (voir la [doc](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html) pour plus d'info):
* range, ip, completion, search_as_you_type, geo_point, geo_shape.

## [Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-overview.html)

> Text analysis is the process of converting unstructured text, like the body of an email or 
> a product description, into a structured format that’s optimized for search.

Les 2 plus commun sont `keyword` et `text` (les 2 utilisé dans le mapping).

1. [`keyword`](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html) (Analyzer `keyword`):
   
Ne fait aucune analyze. Il prendre le texte comme il est.  Pratique pour des id, courriel, code postal, tag.

Exemple: si mon champs contient `Hello World!` ma recherche doit être exactement `Hello World!`, majuscule, espaces, etc.

```bash
POST _analyze
{
  "analyzer": "keyword",
  "text": "Hello World!"
}
---
Hello World!
```

2. [`text`](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html) (Analyzer [`standard`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-analyzer.html)):
   L'analyzer `standard` est celui de base. Il décompose en mot avec les espaces, enlève la ponctuation, met tout en minuscule.
   
```bash
POST _analyze
{
  "analyzer": "standard",
  "text": "Hello World!"
}
---
[hello world]
```

Lors d'une recherche sur un champ analyzer, l'input est également analyzer avec le même analyzer. 

Pour la suite, voir la [doc](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html). 
On peut configurer un analyzer, en créer, etc...

Voici une liste de concept qu'on peut y retrouver:
* Choisir la langue, file path, pattern tokenizer (regex), n-gram (début de mot), keep types (garder les mot (défaut) + ponctuation), synonym, phonetic, reverse, unique

## [Query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)

On a vu comment créer un indice avec un mapping, définir le type, sont analyzer. On peut donc commencer à chercher.

### Query vs Filter

[Rappel](https://petalmd.github.io/chapter-rails-blog/reunion/2021/2021-01-28.html#elasticsearch-query-filter-score)

Les query n'impacte pas le score du match dans la recherche, seul les filter joue sur le scope.

### [Full text queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/full-text-queries.html)

1. [Match](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html)

C'est le plus basic. La l'input est analyzer avec le même analyzer que le field ou l'analyzer par défaut de l'index.

2. [Match phrase prefix query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query-phrase-prefix.html)

Le document doit contenir le text sauf le dernier mot qui est concidérer comme un prefix.

```bash
GET /books/_search
{
  "query": {
    "match_phrase_prefix": {
      "title": "lord of the r"
    }
  }
}
```

2. [Multi-match](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html)

Un match mais sur plusieurs fields.

### [Term-level queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/term-level-queries.html)

Ce base sur les données précis. L'input n'est pas analyzer. Pratique pour un range de date, recherche par id, tag, boolean, etc.

1. [Term query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-term-query.html)

Trouve les documents qui containt le terme **exacte**. Utiliser [`terms`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-terms-query.html)
Pour avoir exactement 1 ou plusieurs termes qui match.

1. [Prefix query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-prefix-query.html)

Trouve les documents qui containt un prefix précis.

## [Aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)

Format les données pour générer des metric, statistic ou autres. Les graphiques, dashboards de Kibana utilise cette feature.

Exemple: group by, maximum, average, histogramme interval de temps 

```bash
GET /books/_search
{
  "aggs": {
    "by_genres": {
      "terms": { "field": "genres" }
    }
  }
}

GET /books/_search
{
  "aggs": {
    "by_genres": {
      "terms": {
        "field": "genres",
        "size": 10
      },
      "aggs": {
        "hits": {
          "top_hits": {
            "size": 10
          }
        }
      }
    }
  }
}
```

## ElasticSearch et Rails

Petit app Rails pour montré les différentes concept que représente l'intégrtion de Elasticsearch et de Rails.

Ce que ça prend:

1. Avoir un HTTP client pour appeler le RESTful API d'elasticsearch, exemple HTTParty
2. Créer le mapping pour l'indice Book, exemple dans une migration: [db/migrate/20210206015609_create_books_index.rb](https://github.com/Bhacaz/book_store_es/blob/main/db/migrate/20210206015609_create_books_index.rb)
3. Job async (idéalement) pour updater les documents: [app/jobs/reindex_job.rb](https://github.com/Bhacaz/book_store_es/blob/main/app/jobs/reindex_job.rb)
4. `IndexableConcern` qu'on peut ajouté a un model pour ajouter des méthodes relatif aux indices, comme `update_index`, `index_name` `recherche`: [app/models/concerns/indexable_concern.rb](https://github.com/Bhacaz/book_store_es/blob/main/app/models/concerns/indexable_concern.rb)
5. Définir un mode de serialization qui match le mapping de l'index (raccourci, overwrite `to_json`) [app/models/book.rb](https://github.com/Bhacaz/book_store_es/blob/main/app/models/book.rb)
6. Utiliser ES pour le recherche: [app/controllers/books_controller.rb#L9](https://github.com/Bhacaz/book_store_es/blob/main/app/controllers/books_controller.rb#L9)

Pour un cas de vrai app qui utilise quelque chose d'aussi simple vous pouvez aller voir: [forem/forem](https://github.com/forem/forem/blob/master/app/models/concerns/searchable.rb)
c'est l'app Rails derière [dev.to](https://www.dev.to).

## [Chewy](https://github.com/toptal/chewy)

La gem permet de gérer beaucoup de chose.

- Le client HTTP (qui prend en fait la gem officiel d'Elasticsearch)
- Mapping des indices
- Serialization via la définition du mapping (clever)
- Manipulation CRUD des index
- Action CRUD des documents
- Stratégie d'indexation, sidekiq, inline
- DSL pour la recherche
- Document elasticsearch dans un object Ruby

### Définir un indice

[app/chewy/accounts_index.rb](https://github.com/petalmd/petalmd.rails/blob/master/app/chewy/accounts_index.rb)

```ruby
class AccountsIndex < Chewy::Index
  define_type Account do
     field :id, type: :keyword
     field :first_name, type: :text
     field :last_name, type: :text
     field :email, type: :keyword
  end
end
```

### Manipulation des indices

```ruby
# Créer l'index si il n'existe pas
AccountsIndex.create
# Supprimer si il existe
AccountsIndex.delete
# Supprimer et recréer
AccountsIndex.purge
```

### Manipulation des documents

```ruby
# Import tout, ou les ids en paramètre
AccountsIndex.import
AccountsIndex.import [1, 2, 3]
# Supprimer l'index, créer l'index et réimporte tout
AccountsIndex.reset
```

### Strategy

Définie de quel façon les index ce mettre à jour, sidekiq VS inline.

```ruby
class Account < ApplicationRecord
   update_index('AccountsIndex') { self }
end
```

`update_index` rajoute un `after_commit` pour appeler `AccountsIndex.import self.id`, mais ce `import` peut être appeler inline ou dans un worker.
Par défaut (dans notre app) c'est dans un worker. Si un account update son prénom, il faut pratiquement update tout les indexs (events, appointments, account, membership).
Pour ne allonger le call de l'API un worker est créer pour chaque chaque 100 documents pour chaque index (#parallelisme). Si on ne veut
attendre après Sidekiq on peut utiliser la strategy atomic (par défaut dans la console rails).

```ruby
Chewy.strategy(:atomic) do
   Account.first.update(first_name: 'Hello')
end
```

À la fin du _block_ tout les indexs seront à jour.

### DSL recherche

Plus haut je n'est pas parler des `bool`, `should`, `must` dans les query.
Cela permet d'enchainés plusieurs query un peut comme plusieurs `where` pour ActiveRecord.

```bash
GET books/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "genres": {
              "value": "Fantasy"
            }
          }
        },
        {
          "term": {
            "title": {
              "value": "ring"
            }
          }
        }
      ]
    }
  }
}
```

```ruby
BooksIndex.query(term: { 'genres' => 'Fantasy'} ).query(math: { 'title' => 'ring' })
```

```ruby
BooksIndex::Book.agg(term: { 'by_name' => { term: {field: 'first_name'}}})
```


https://www.elastic.co/guide/en/elasticsearch/reference/7.10/query-dsl-intervals-query.html


