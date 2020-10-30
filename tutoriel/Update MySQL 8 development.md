---
layout: default
title: Update MySQL 8 development
parent: Tutoriel
nav_order: 2
---

# Update MySQL 8 development

## 5.6 => 8.0

Arrêter les containers

```bash
dcd
# ou
docker-compose -f ~/code/petalmd.rails/docker-compose.yml down
```

Modifier le docker-compose.yml

```diff
# ./docker-compose.yml

- image: "mysql:5.6"
+ image: "mysql:8.0"
```

Installer le client MySQL 8

```bash
brew install mysql@8.0
brew link --overwrite mysql@8.0
```

Il est possible que `openssl` s'update. Si vous rencontrer des problèmes réinstaller la gem `mysql2`

```bash
gem uninstall mysql2
bundle config --local build.mysql2 "--with-opt-lib=/usr/local/opt/openssl/lib"
bundle install
```

Démarrer les containers

```bash
dcu
# ou
docker-compose -f ~/code/petalmd.rails/docker-compose.yml up -d
```

Lancer un `dbr`

```bash
dbr
```

Pour valider, vous pouvez ouvrir une console rails

```ruby
>> mysql8? # => true
```


## 8.0 => 5.6

Arrêter les containers

```bash
dcd
# ou
docker-compose -f ~/code/petalmd.rails/docker-compose.yml down
```

Modifier le docker-compose.yml

```diff
# ./docker-compose.yml

- image: "mysql:8.0"
+ image: "mysql:5.6"
```

Changer la version linker du client mysql par brew

```bash
brew unlink mysql && brew unlink mysql@5.6 && brew link --force mysql@5.6
```

Démarrer les containers

```bash
dcu
# ou
docker-compose -f ~/code/petalmd.rails/docker-compose.yml up -d
```

Lancer un `dbr`

```bash
dbr
```

Pour valider, vous pouvez ouvrir une console rails

```ruby
>> mysql8? # => false
```

