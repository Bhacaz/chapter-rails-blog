---
layout: default
title: 22 octobre 2020
parent: 2020
grand_parent: Réunion
nav_order: 19
---

# 22 octobre 2020

## Tour de table

## Nested transaction

```
 bundle exec rails g model Test text:string
```

### Petit rappel

Une transaction permet de cumuler les opérations à effectuer dans la DB et faire
certain lock jusqu'à la fin du bloc. Cela permet de vérifier que
tout est valide. Si un erreur survient dans le block, les opérations peuvent être revert
a partir du point de sauvegarde.

Particularier, l'erreur `ActiveRecord::Rollback` qui est lancer dans
une transaction ne raise pas réellement d'erreur. La transaction sera rollbacker
et le l'exécution va se poursuivre.

```ruby
ActiveRecord::Base.transaction do
  Test.create! text: 'Hello'
  raise ActiveRecord::Rollback
end

puts 'Hello'
```

### Expérimentation

Dans le cas ou on a plusieurs transaction imbriqué, comment les rollbacks 
sont gérés?

```ruby
ActiveRecord::Base.transaction do
  Test.create! text: 'Hello'
  ActiveRecord::Base.transaction do
    Test.create! text: 'World'
    raise ActiveRecord::Rollback
  end
end

Test.count # => 2 😱
```

Le pourquoi 2 records sont créés est que la transaction parent ne
voit pas le `raise ActiveRecord::Rollback` qui est _catch_ pas la transaction
enfant. Ce qui implique que la transaction parent qui cumule les opérations
 va être commiter et inclue les opérations de la transaction
enfant.

Pour contourner le problème, il est possible de _require_ une nouvelle
transaction `ActiveRecord::Base.transaction(requires_new: true)`. En MySQL
cela va créer un _savepoint_ qu'on pourra peut utiliser comme point de sauvegarde pour rollbacker.

```ruby
ActiveRecord::Base.transaction do
  Test.create! text: 'Hello'
  ActiveRecord::Base.transaction(requires_new: true) do
    Test.create! text: 'World'
    raise ActiveRecord::Rollback
  end
end
#   (2.2ms)  BEGIN
#  Test Create (1.8ms)  INSERT INTO `tests` (`text`, `created_at`, `updated_at`) VALUES ('Hello', '2020-10-19 20:00:16', '2020-10-19 20:00:16')
#   (1.7ms)  SAVEPOINT active_record_1
#  Test Create (2.5ms)  INSERT INTO `tests` (`text`, `created_at`, `updated_at`) VALUES ('World', '2020-10-19 20:00:16', '2020-10-19 20:00:16')
#   (4.9ms)  ROLLBACK TO SAVEPOINT active_record_1
#   (2.3ms)  COMMIT
#   (1.5ms)  SELECT COUNT(*) FROM `tests`

Test.count
# => 1
```

Pour plus de détail voir la documentation 
officiel de Rails sur le suject: [Nested transactions](https://api.rubyonrails.org/classes/ActiveRecord/Transactions/ClassMethods.html#module-ActiveRecord::Transactions::ClassMethods-label-Nested+transactions)

Pour plus de détail: [surprises-with-nested-transactions-rollbacks-and-activerecord](https://pragtob.wordpress.com/2017/12/12/surprises-with-nested-transactions-rollbacks-and-activerecord/
)

## Opérations multiple dans une transaction

Modèle de test

```ruby
class Test < ApplicationRecord
  after_update_commit :updated_method
  after_destroy_commit :destroy_method

  def updated_method; p 'updated'; end

  def destroy_method; p 'destroyed'; end
end
```

Un problème fut trouvé lorsqu'on effectue des opérations (update, destroy)
sur 2 instance du même record.

```ruby
Test.last.update! text: 'abc' # => updated
Test.last.destroy! # => destroyed
```

Dans l'example plus haut 2 _commit_ sera effectuer et les callbacks seront
exécuter.


```ruby
ActiveRecord::Base.transaction do
  Test.last.update! text: 'abc'
  Test.last.destroy!
end
# => updated
```

Dans l'example seulement le `after_update_commit` sera effectuer et
dans le context de l'instance tu premier `Test.last`. Ce qui implque 
que à la fin du block, `Test.last` est bien supprimer de la db, mais
l'instance au moment on exécute `after_update_commit` ne le sais pas.
Autrement dit `destroyed? # => false`. Plein de problème peut en découler.

Le problème provient que dans un transaction, Rails stock tout les records
modifier dans un array et fait `Array#uniq` avant d'appeler les callbacks.
Puisque qu'un activerecord est uniq selon sa class et sont id seulement
le premier record exécute ses callbacks. 

C'est corriger en Rails 6 [rails/rails#36190](https://github.com/rails/rails/pull/36190)

## `||=`

```ruby
>> a = false
false
>> a ||= 123
123
>> a
123
```

🤦‍♂️

`||=` est un racourcie pour `a || a = b` donc si `a # => false` on
éxécute la partie de droite, logique...

Pour plus de détails: [what-rubys-double-pipe-or-equals-really-does](http://www.rubyinside.com/what-rubys-double-pipe-or-equals-really-does-5488.html
)

## Benchmark d'API `ab`

Je l'ai utilisé quelque fois. 
C'est juste pour voir dire que sa existe. Et c'est déjà installer 
sur mac os.

[ab - Apache HTTP server benchmarking tool
](https://httpd.apache.org/docs/2.4/programs/ab.html)

Example:

```
ab -H 'Authorization: Bearer <<token>>' \
-c 1 -n 100 \
https://app.stg3.stg.petalmd.com/api/v2.1/hospitals/1317/dashboards/console_layout_rows\?hospitalId\=1317\&startTime\=2020-10-20\&endTime\=2020-10-26\&page\=1\&perPage\=150\&layoutId\=1071\&searchInitials\=false
```

```
This is ApacheBench, Version 2.3 <$Revision: 1843412 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking app.stg3.stg.petalmd.com (be patient).....done


Server Software:        PetalMD
Server Hostname:        app.stg3.stg.petalmd.com
Server Port:            443
SSL/TLS Protocol:       TLSv1.2,ECDHE-RSA-AES256-GCM-SHA384,4096,256
Server Temp Key:        ECDH X25519 253 bits
TLS Server Name:        app.stg3.stg.petalmd.com

Document Path:          /api/v2.1/hospitals/1317/dashboards/console_layout_rows?hospitalId=1317&startTime=2020-10-20&endTime=2020-10-26&page=1&perPage=150&layoutId=1071&searchInitials=false
Document Length:        365340 bytes

Concurrency Level:      1
Time taken for tests:   93.507 seconds
Complete requests:      100
Failed requests:        0
Total transferred:      36574100 bytes
HTML transferred:       36534000 bytes
Requests per second:    1.07 [#/sec] (mean)
Time per request:       935.068 [ms] (mean)
Time per request:       935.068 [ms] (mean, across all concurrent requests)
Transfer rate:          381.97 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       78   98  47.1     91     481
Processing:   580  837 201.7    792    1805
Waiting:      426  687 191.9    659    1628
Total:        679  935 219.0    886    1886

Percentage of the requests served within a certain time (ms)
  50%    886
  66%    960
  75%    999
  80%   1067
  90%   1198
  95%   1391
  98%   1826
  99%   1886
 100%   1886 (longest request)
```

A noté que `localhost` fonctionne pas. Utliser `127.0.0.1`.

## HTTP Header et encodage

[petalmd/petalmd.rails#6108](https://github.com/petalmd/petalmd.rails/pull/6108)

## `merge` ActiveRecord::Relation

[ActiveRecord::SpawnMethods#merge](https://api.rubyonrails.org/classes/ActiveRecord/SpawnMethods.html#method-i-merge)
 
 ```ruby
Post.where(published: true).joins(:comments).merge( Comment.where(spam: false) )
# Performs a single join query with both where conditions.

recent_posts = Post.order('created_at DESC').first(5)
Post.where(published: true).merge(recent_posts)
# Returns the intersection of all published posts with the 5 most recently created posts.
# (This is just an example. You'd probably want to do this with a single query!) 
```