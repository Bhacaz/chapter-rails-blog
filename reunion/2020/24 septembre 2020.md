---
layout: default
title: 24 septembre 2020
parent: 2020
grand_parent: Réunion
nav_order: 17
---

# 24 septembre 2020

## Tour de table

## [Hacktoberfest](https://hacktoberfest.digitalocean.com/details#quality)

J'en parle parce que ça donne une bonne raison de s'y intéressé et 
de participer au projet open source. Et on en utilise beaucoup.

Il suffit simplement d'ouvrir 4 Pull Request durant le mois d'octobre. Pas besoin qu'elle soit mergé,
ni que se soit un gros project, cela peut être un repo public personnel. Il faut s'inscrire ici [hacktoberfest.digitalocean.com](https://hacktoberfest.digitalocean.com/) pour avoir
droit au t-shirt et collant si vous compléter le défi.

### Idée de projet

* [petalmd/bright_serializer](https://github.com/petalmd/bright_serializer)
    * J'ai créé 3 issues pour données des idées.
* [ruby-faker/faker](https://github.com/faker-ruby/faker)
    * Vous pouvez juste traduire des trucs en `fr_CA` par example.
* Les labels `hacktoberfest` sur des issues un peu partout sur Github.

## Le body dans les tests d'API

Avec l'API on travaille toujours en JSON... sauf dans les tests.

Nouha est tombé sur un problème [petalmd.rails#5978 (comment)](https://github.com/petalmd/petalmd.rails/pull/5978#issuecomment-696070726) lors de l'écrirture d'un test, bien que avec Postman tout fonctionnait bien.

### Rack::Test

```ruby
# rack-test-1.1.0/lib/rack/test.rb:265

def process_request(uri, env)
  p env['CONTENT_TYPE'] # => "application/x-www-form-urlencoded"
  p env['rack.input'].read() # => "group_id=273&web_offers[][activated]=true&web_offers[][grid][][cwday]=2&web_offers[][grid][][web]=true&web_offers[][grid][][start_moment]=600&web_offers[][grid][][end_moment]=660&web_offers[][account_id]=174&web_offers[][activated]=true&web_offers[][grid][][cwday]=3&web_offers[][grid][][web]=false&web_offers[][grid][][start_moment]=600&web_offers[][grid][][end_moment]=660&web_offers[][grid][][cwday]=4&web_offers[][grid][][web]=true&web_offers[][grid][][start_moment]=600&web_offers[][grid][][end_moment]=660&web_offers[][account_id]=175&web_offers[][activated]=true&web_offers[][grid][][cwday]=6&web_offers[][grid][][web]=true&web_offers[][grid][][start_moment]=900&web_offers[][grid][][end_moment]=1020&web_offers[][grid][][cwday]=7&web_offers[][grid][][web]=true&web_offers[][grid][][start_moment]=900&web_offers[][grid][][end_moment]=1080"

  @rack_mock_session.request(uri, env)      
  
  ...
end
```

Quand on ajoute un body a un `POST, PATC, PUT...` le content-type par défaut est `"application/x-www-form-urlencoded"`.
Source content-type défaut: `env['CONTENT_TYPE'] ||= 'application/x-www-form-urlencoded'` `rack-test-1.1.0/lib/rack/test.rb:243`


Et c'est la méthode `Rack::Test::Utils#build_multipart` qui transforme le Hash passer dans test en `x-www-form-urlencoded`.
Cette méthode peux transformer assez votre Hash pour faire failed votre test car dans la norme chaque objet doit avoir la même signature.

Exemple:

```ruby
[
   {
    id: 1,
    middle_name: 'Abc'
  },
  {
    id: 2,
    middle_name: 'Efg'
  }
]
```

Si on veut envoyer:

```ruby
[
   {
    id: 1,
    middle_name: 'Abc'
  },
  {
    id: 2
  }
]
```

Les params reçu dans Grape après avoir passé par Rack et `Rack::Test::Utils#build_multipart` va ressembler à ceci.

```ruby
[
   {
    id: 1,
    middle_name: 'Abc'
  },
  {
    id: 2,
    middle_name: nil # <===
  }
]
```

Ce qui est pas la même chose surtout si on demande un params `optional :middle_name, allow_blank: false`, ça ne passe 
pas la validation.

### Solution

On peut facilement spécifier le content-type de notre request dans les tests avec `as: :json`.

```ruby
patch api_path, params: params, headers: headers, as: :json
```

## Remplacer Codecov

Il reste environ 1-2 semaines avec notre contract avec Codecov, il faudrait le remplacer. Si vous avez des idées
solution, produit, expérience passé. Je crois que la feature clé c'est le [Codecov Delta](https://docs.codecov.io/docs/codecov-delta)
qui calcule le coverage sur les nouvelles lignes.

J'ai fais une [preuve de concept](https://github.com/Bhacaz/bright_serializer/pull/1) d'une solution maison. 
C'est entre le c'est assez facile pour la finir et l'utiliser
et compliqué pour dire qu'on veut pas maintenir cela et qu'on est mieux de payer un service.

Des suggestion et avis???

