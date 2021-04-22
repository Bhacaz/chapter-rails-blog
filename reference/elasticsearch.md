---
layout: default
title: Elasticsearch
parent: Référence
nav_order: 4
---

# Elasticsearch

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

<details>

<summary>Exemple</summary>

{% highlight ruby %}
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

{% endhighlight %}

</details>

## Aggregations

Utiliser `collapse` pour retourner des distinct value plutôt que une aggregation `terms`.

https://www.elastic.co/guide/en/elasticsearch/reference/6.8/search-request-collapse.html#search-request-collapse
https://discuss.elastic.co/t/how-to-return-distinct-values-from-query-based-on-a-field/228000/2