---
layout: default
title: Updater Ruby
parent: Tutoriel
nav_order: 1
---

# Updater Ruby

## Installer la nouvelle version
```bash
brew update && brew upgrade ruby-build
rbenv install 2.6.6 
```

_openssl et ruby-2.6.6 peut être très long._

## Utiliser la nouvelle version

```bash
ruby --version
# => 2.6.6
```

Devrait être le même que dans le fichier `.ruby-version`

On peut aussi le mettre global

```bash
rbenv global 2.6.6
```

## Installer bundler et gems

[PetalMD Wiki](https://github.com/petalmd/documentation/wiki/Developer-Setup#install-be-gems)

```bash
gem install bundler --version 1.17.3
bundle config --local build.mysql2 "--with-opt-lib=/usr/local/opt/openssl/lib"
bundle install
git checkout .bundle/config
```

## Cleanup

**Voir les versions installées**

```bash
rbenv versions
```

**Pour supprimer les vielles verisons**

```bash
rbenv uninstall 2.4.9
```

