# Elasticsearch Exact match

We add the mapping for the fields with special configurations only.

For exact match, use normalizer with lowercase filter also can add filter to convert characters automatically such as double space **\\u0020\\u0020** to one space **\\u0020** and **#** to word **number**.

example create index "book" with type "page" and tow fields "text" and "line", we might have some other fields such as page number, line number but we are intended to use the default congiuration for them and hence Elasticsearch will configure them during indexing (data loading).
```
PUT /book
{
  "settings": {
    "analysis": {
        "char_filter": {
            "wc_filter": {
                "type": "mapping",
                "mappings": [
                  "\\u0020\\u0020 => \\u0020",
                  "# => number"
                ]
            }
        },
        "normalizer": {
            "wc_normalizer": {
                "type": "custom",
                "char_filter": ["wc_filter"],
                "filter": ["lowercase"]
            }
        }
    }
  },
  "mappings": {
    "page": {
        "properties": {
            "text": {
                "type": "text",
                "fields": {
                    "keyword": { 
                        "type": "keyword",
                        "normalizer": "wc_normalizer"
                    }
                }
            },
            "line": {
              "type": "text",
                "fields": {
                    "keyword": {
                        "type": "keyword",
                        "normalizer": "wc_normalizer"
                    }
                }
            }
        }
    }
  }
}
```

Sample query, find exact match of text "William Shakespeare" so we use the match-phrase query:
```
GET /book/_search
{
  "query": {
    "match_phrase": { "text.keyword": "William Shakespeare"}
  }
}
```
**OR**
```
GET /book/_search
{
  "query": {
    "match_phrase": { "text.keyword": "william shakespeare"}
  }
}
```
Also WILLIAM SHAKESPEARE will yield the same result, since no case-sensitivty with lowercase filtering.

Another good use-case if we need 
- exact result
if not,
- nearst match

the scoring is boosted if there is exact match.

Using nested queries "bool"
should -> OR
```
GET /book/_search
{
  "from": 0,
  "query": {
    "bool": {
      "should": [
        {"bool":{
          "should": [
            {"match_phrase": { "text.keyword": "William Shakespeare"}},
            {"match_phrase": { "line.keyword": "William Shakespeare"}}
          ], "boost": 100
        }},
        {"bool": {
          "should": [
            {"match": { "text": "William Shakespeare"}},
            {"match": { "line": "William Shakespeare"}}
          ]
        }} 
      ]
      
    }
  },
  "size": 2000
}
```
"boost" score by *100* for exact match (match-phrase) otherwise use normal match query.
