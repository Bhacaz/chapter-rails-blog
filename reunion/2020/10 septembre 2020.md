---
layout: default
title: 10 septembre 2020
parent: 2020
grand_parent: Réunion
nav_order: 16
---

# 10 septembre 2020

## Tour de table

## [ActionMailer](https://github.com/petalmd/petalmd.rails/pull/5959)

Cette fonctionnalité de Rails permet d'envoyer email à l'aide des vues Rails.

On commence a l'utiliser parce que la gestion dans Mandrill commence a être compliquer et très mal adapter.
Beaucoup de templates, dupliquer en/fr, éditeur moyen, logique compliquer a faire directement dans le template, pas de versionnering,
oublie de sync entre l'environnment de test et de prod etc...

Créer les vues de la même façon que le fait dans rails habituellement et utiliser toutes les fonctionnalité: 
    
* layout, view, partiel, stylesheet, assets, erb, I18n, etc

### Structure

* `app/mailers/application_mailer.rb`
    * Le ActionMailer de base avec les paramètres par défaut, comme `ApplicationController`
* `app/mailers/schedule/event_mailer.rb`
    * L'équivalent d'un `Controller` avec différentes `actions` qui représente les différentes emails.
    * On pourrait avoir par example un `AccountMailer` avec `change_password`, `confirme_email`, etc
* `app/views/layouts/mailer/schedule.html.erb`
    * Le layout pour les emails de schedule (booking a un autre style). C'est la coquille du email (header, footer, style). 
    On ne devrait pas avoir besoin de le changer très souvent.
* `app/views/schedule/event_mailer/request_change_owner.html.erb`
    * View qui contient le content du emails
* `spec/mailers/previews/schedule/event_mailer_preview.rb`
    * Une des meilleur features, la possibilter de voir les templates avec des données déjà prés remplie

### Utilisation

Pour envoyer un mail il suffit d'appeler l'action: 

```ruby
Schedule::EventMailer.with(account: Account.last).request_change_owner.deliver_later
```

* `with`: permet de passer des éléments à `params` (comme dans un controller)
* `request_change_owner`: correspond à l'action et à la vue `app/views/schedule/event_mailer/request_change_owner.html.erb`
* `deliver_later`: permet d'envoyer le email via Sidekiq. `deiliver_now` est également disponible.

**Environment**

* Development: 
    * Par défaut, les emails sont enregistrer dans `tmp/mailers` sous la forme de fichier texte.
    * On peut envoyer réllements des emails en décommentent `MAILER_SMTP_PASSWORD` dans `application.yml`. 
    Mais les assets ne foncitonne pas puisqu'ils sont hoster sur `localhost:3000` qui n'est pas accessible via Gmail
* Test: Les _object_ ActionMailer sont enregistrer dans `ActionMailer::Base.deliveries`
* Staging, Production: Les emails sont envoyer via Mandrill a l'aide de leur serveur SMTP
 
### Preview

La liste de tout les emails sont disponible ici http://localhost:3000/rails/mailers il revient au dev de passer les données.

C'est disponible en:

* Dévelopmment (utile avec le autoreload si on travaille le visuel)
* Test `RAILS_ENV=test bin/rails server`. Utile pour générer les data avec `FactoryBot` mais il y a pas de autoreload.


### Rspec
C'est possible de tester mais j'ai pas beaucoup exploré.

https://relishapp.com/rspec/rspec-rails/v/3-8/docs/mailer-specs

## [Enumerators in Ruby](https://makandracards.com/makandra/35993-enumerators-in-ruby)
