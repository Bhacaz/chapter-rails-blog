---
layout: default
title: Performance
parent: Référence
nav_order: 3
---

# Performance

La quesiton de performance est importante dans le processus dévelopmment et devrait faire partie des éléments
a observer dans les reviews. Voici une liste d'outils pour comparer plusieurs implémentations.

## Référence

* [Fast Ruby](https://github.com/JuanitoFatas/fast-ruby).
Repository grandement cité chez les Rubyist. 
Plusieurs cops de RuboCop Performance sont basé sur ces Benchmarks.

## Outils

### Static

* [RuboCop Performance](https://docs.rubocop.org/rubocop-performance/)
* [Bullet](https://github.com/flyerhzm/bullet).

### Benchmark

* [benchmark-ips](https://github.com/evanphx/benchmark-ips).

```ruby
require 'benchmark/ips'
Benchmark.ips do |x|
  x.report('A') { }
  x.report('B') { }
  x.compare!
end
```

### Profiler

* [MemoryProfiler](https://github.com/SamSaffron/memory_profiler).

```ruby
require 'memory_profiler'
report = MemoryProfiler.report(allow_files: 'petalmd.rails') { }
report.pretty_print(to_file: 'profiling.txt')
```

* [benchmark-memory](https://github.com/michaelherold/benchmark-memory)

```ruby
require "benchmark/memory"
Benchmark.memory do |x|
  x.report('A') { }
  x.report('B') { }
  x.compare!
end
```
