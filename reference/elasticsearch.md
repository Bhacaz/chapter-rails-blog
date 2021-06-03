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

Le type `keyword` est le type le plus performant. Puisqu'on pratiquement jamais de query _range_, _greater than_ mais juste des filters
`terms` le plus efficace c'est de les mapper en `keyword`.

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

### Multiple fields

Il peut y avoir plusieurs sub-fields différents qui répond a des besoin différents. Un bon et vrai cas d'utilisation serait grouper les
noms de famille par première lettre de nom de famille.

* last_name.raw => keyword (Dubé)
* last_name.text_search => standard_ascii_lower (dube)
* last_name.first_letter => first_letter_keyword (d)

Ça permetterait de faire une filtre ou une aggregation très performant (à cause de l'analyzer keyword) sur un field specifique dédier. 

### Choix de l'analyzer

En règle général l'analyzer [`standard`](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/analysis-standard-analyzer.html)
est un excellent choix. Dans notre code il a été bonifier et est utilisable via **`standard_folding`**. C'est basé
sur le `standard` fournie par ES, mais qui convertie les caractères non ascii (`é` => `e`), pratique en français.

Les analyzers souvent utilisé dans le cluster ES1 `folding`, `minimal_folding` et `folding_ngram` son coûteux au moment de
la réindexation et ne respecte pas les paramètres par défaut d'ES pour la création d'analyzer. C'est un béquille qui
sert à chercher au début d'un mot. Ce qui peut facilement être fait avec une query `prefix`. Ils sont à éviter si possible.
Seul la cas où on veut chercher au milieu d'un mot peu justifier d'utilisé `folding_ngram`, c'est rare et une [query string](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html)
avec un wildcard au début est possible, mais peu performant. 

La performance à la recherhe peut être améliorer avec [`index_prefixes`](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/index-prefixes.html).

L'utilisation du type [`search_as_you_type`](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/search-as-you-type.html) 
peut également être envisager. On en root on a l'analyzer spécifier, et des sub-fields automatiquement générerr avec du ngram et de la cache de prefix.

### Searchable (experimental)

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

## Aggregations

Utiliser `collapse` pour retourner des distinct value plutôt que une aggregation `terms`.

https://www.elastic.co/guide/en/elasticsearch/reference/6.8/search-request-collapse.html#search-request-collapse
https://discuss.elastic.co/t/how-to-return-distinct-values-from-query-based-on-a-field/228000/2
