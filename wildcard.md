# Elasticsearch Wildcard exact match

We add the mapping for the fields with special configurations only.

For willdcard exact match, use normalizer with lowercase filter

example create index "book" with type "book" and one field "text", we might have some other fields such as page number, line number but we are intended to use the default congiuration for them and hence Elasticsearch will configure them during indexing (data loading).
```
PUT book
{
  "book": {
    "settings": {
      "analysis": {
         "normalizer": {
            "case_insensitive": {
              "type": "custom",
              "tokenizer": "keyword",
              "filter": [
                "lowercase"
              ]
            }
         }
      },
      "number_of_shards": "1",
      "number_of_replicas": "0"
   },
    "mappings": {
      "text": {
        "properties": {
          "fields": {
            "keyword": {
              "type": "keyword",
              "normalizer": "case_insensitive"
            }
          }
        }
      }
    }
  }
}
```

Sample query, find exact match of text "the goal is to search within many book text" so we use the match-phrase query:
```
GET /book/_search?pretty
{
  "query": {
    "wildcard": {
              "text.keyword": {
                  "value":"*the goal is to search within many book text*",
                  "boost": 20
              }
    }
  }
}
```

use the **\*** as a wildcard to surround the text, the text can be small or capital case since no case-sensitivty with lowercase filtering.
