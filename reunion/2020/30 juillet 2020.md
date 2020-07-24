---
layout: default
title: 30 juillet 2020
parent: 2020
grand_parent: Réunion
nav_order: 14
---

# 30 juillet 2020

## Tour de table

## VCR 📼

Gem d'on j'ai entendu parlé qui pourrait être intéressant en autre dans booking.
Du moins il est intéressant de savoir que sa existe.

Nous avons déjà `webmock` pour éviter les call http durant les tests. Sinon sa
serait trop long et on pourrait tomber sur des erreurs de rate limit.

La gem [vcr](https://github.com/vcr/vcr) permet de mocker facilement ces appel
en les exécutant pour de vrai si la cassette n'existe pas, l,enregistrant dans un fichier `.yml`
et quand on veut mocker la même requête on rejoue la cassette.

### Example

Je veux récupérer l'information des users avec l'api Github.

```ruby
def getGithubProfilePicture(profile)
  response = HTTParty.get("https://api.github.com/users/#{profile}")
  JSON.parse(response)['avatar_url']
end
```

Aujourd'hui on pourait faire quelque chose du genre:

```ruby
let(:profile) { 'petalmd' }
it 'return the avatar_url' do
  allow(HTTParty).receive(:get).with("https://api.github.com/users/#{profile}") { {'avatar_url' => 'example.com'}.to_json }
  expect(getGithubProfilePicture(profile)).to include('example.com')
end
```

Si dans 1 mois Github changait `avatar_url` pour `avatar_uri` notre CI le trouverais pas.
Ou si un DME changait la signature de leur API même chose.

### Example complet avec VCR

```ruby
require 'httparty'
require "vcr"

VCR.configure do |c|
  c.cassette_library_dir = "spec/vcr"
  c.hook_into :webmock
  c.ignore_localhost = true
end

describe 'Test VCR' do
  def getGithubProfilePicture(profile)
    response = HTTParty.get("https://api.github.com/users/#{profile}")
    JSON.parse(response)['avatar_url']
  end

  let(:user_response) do
    VCR.use_cassette("github/user", re_record_interval: 1.week.to_i) { getGithubProfilePicture('petalmd') }
  end

  it { expect(user_response).to match(URI::regexp) }
end
```

L'intéret que je vois c'est de mêtre l'options `re_record_interval` par example 
1 fois semaine pour valider que la signature des appels d'API est la même et qu'on
l'identifie dans le CI.

**Ça vous parle?**

