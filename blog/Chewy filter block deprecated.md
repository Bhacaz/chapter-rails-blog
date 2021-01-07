---
layout: default
title: Chewy `filter(&block)` deprecated
parent: Blog
nav_order: 8
---

# Chewy `filter(&block)` deprecated

On utilise présentement la version [Legacy du DSL](https://github.com/toptal/chewy/blob/master/LEGACY_DSL.md) 
pour construire des query Elasticsearch de Chewy. Pour la documentation
de la nouvelle version voir [toptal/chewy README.md#search-requests](https://github.com/toptal/chewy#search-requests).

Pour la transition, le plus simple est d'utiliser des hash qui 
a la même structure qu'une query native Elasticsearch, plutôt que le DSL. Voici une procédure pour faire la converstion.

## Procédure

```ruby
# app/services/common/search_service.rb:240

def search_memberships(query, group_id_param)
  raise ArgumentError unless group_id_param.present? && group_id_param.is_a?(Integer)

  scope = MembershipsIndex.query \
    multi_match: {
      fields: %w(*_name *_name.folded license),
      type: :phrase_prefix,
      query: query
    }
  scope = scope.filter { group_id == group_id_param } # <=== Block to convert

  scope.order('last_name.folded', 'first_name.folded')
end
```

Avec un objet `Chewy::Query` on peut générer et récupérer le hash pour facilement convertir le block en hash.

```ruby
Common::SearchService.search_memberships('nancy', 264)
# => #<MembershipsIndex::Query:0x00007fda607e2c18 @options={}, @_types=[], @_indexes=[MembershipsIndex], @criteria=#<Chewy::Query::Criteria:0x00007fda607e2718 @options={:query_mode=>:must, :filter_mode=>:and, :post_filter_mode=>:and}, @queries=[{:multi_match=>{:fields=>["*_name", "*_name.folded", "license"], :type=>:phrase_prefix, :query=>"nancy"}}], @filters=[{:term=>{"group_id"=>264}}], @post_filters=[], @sort=["last_name.folded", "first_name.folded"], @fields=[], @types=[], @scores=[], @search_options={}, @request_options={}, @facets={}, @aggregations={}, @suggest={}, @script_fields={}>, @_request=nil, @_response=nil, @_results=nil, @_collection=nil>
```

```ruby
>> Common::SearchService.search_memberships('nancy', 264).send(:_request)
{
     :body => {
        :query => {
            :filtered => {
                 :query => {
                    :multi_match => {
                        :fields => [
                            [0] "*_name",
                            [1] "*_name.folded",
                            [2] "license"
                        ],
                          :type => :phrase_prefix,
                         :query => "nancy"
                    }
                },
                :filter => {
                    :term => {
                        "group_id" => 264
                    }
                }
            }
        },
         :sort => [
            [0] "last_name.folded",
            [1] "first_name.folded"
        ]
    },
    :index => [
        [0] "development_memberships"
    ],
     :type => []
}
```

Ce qu'on veut:

```ruby
>> Common::SearchService.search_memberships('nancy', 264).send(:_request)[:body][:query][:filtered][:filter]
{
    :term => {
        "group_id" => 264
    }
}
```

Résultat final:

```ruby
# app/services/common/search_service.rb:240

def search_memberships(query, group_id_param)
  raise ArgumentError unless group_id_param.present? && group_id_param.is_a?(Integer)

  scope = MembershipsIndex.query \
    multi_match: {
      fields: %w(*_name *_name.folded license),
      type: :phrase_prefix,
      query: query
    }
  
  scope = scope.filter(term: { group_id: group_id_param }) # <=== converted

  scope.order('last_name.folded', 'first_name.folded')
end
```
