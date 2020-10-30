---
layout: default
title: 24 octobre 2019
parent: 2019
grand_parent: Réunion
nav_order: 2
---

# 24 octobre 2019

- Tour de table

 ---
- Utilizer size VS count.
- Est-ce qu'on peut enlever la table `notifications` et `deliveries` du dump (équivaut a 8.76 Gb de données). On sauve environ 2h!
```sql
SELECT
TABLE_NAME AS `Table`,
ROUND(((DATA_LENGTH + INDEX_LENGTH) / 1024 / 1024 /1024), 2) AS `Size (GB)`
FROM
information_schema.TABLES
WHERE
TABLE_SCHEMA = "petalmd_development"
ORDER BY
(DATA_LENGTH + INDEX_LENGTH)
DESC;
```

- Repensé les la façon qu'on fait les serializer?
- Team Github Backend
- Params Grape et namespace [Grape documentation](https://github.com/ruby-grape/grape#include-parent-namespaces), [PR diff](https://github.com/petalmd/petalmd.rails/pull/4902/files#diff-f2de8f1752b6f2df73b0c4e755bd69c0R4)

### Script de profiling

```ruby
# $ rails c
`spring stop`
file = 'profiler-reload.txt'
require 'memory_profiler'
report = MemoryProfiler.report(allow_files: 'booking/metrics_api') do
  reload!
end
report.pretty_print(to_file: file)
`mine #{file}`
```

**BEFORE**
```
Total allocated: 37551 bytes (385 objects)
Total retained:  8263 bytes (141 objects)

allocated memory by file
-----------------------------------
     37551  /Users/bhacaz/code/petalmd.rails/app/api/v2/analytics/booking/metrics_api.rb
```
	
**AFTER**
```
Total allocated: 9447 bytes (80 objects)
Total retained:  3679 bytes (29 objects)

	allocated memory by file
-----------------------------------
      9447  /Users/bhacaz/code/petalmd.rails/app/api/v2/analytics/booking/metrics_api.rb
```
* Dernier cleanup de Grape **V1** [Graph kibana](https://bit.ly/2JiVdaq)


<!--stackedit_data:
eyJoaXN0b3J5IjpbMzY0NDYzNDkwLDE4Nzc0MzE5NzUsMTUxNz
I4NDc4MSw0OTIzNDUzNTMsOTAxNDI3MzAzLC0xOTY5NDYyOTUy
LC0xMjMyNDYyMjI1XX0=
-->