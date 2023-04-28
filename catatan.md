
# Instalasi
- Untuk cluster minimum docker resource ram 4gb
- Jalankan perintah `docker-compose up --build`


# Catatan Teori

Catatan teori: https://docs.google.com/document/d/1B-y5YZLa3TrqxlVtaoD8l18vJFwXCKt0/edit?usp=sharing&ouid=102860775893369982383&rtpof=true&sd=true



# Praktik
- Melihat Nodes: http://localhost:9200/_cat/nodes?v&pretty

```
ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
172.20.0.2           29          76  11    0.50    1.07     0.70 dilm      *      es02
172.20.0.4           16          76  11    0.50    1.07     0.70 dilm      -      es01
172.20.0.3           27          76  11    0.50    1.07     0.70 dilm      -      es03
```


- Membuat index
```
PUT belajar_elasticsearch
{
  "settings": {
    "index": {
      "number_of_shards": 3,  
      "number_of_replicas": 1
    }
  }
}
```

- Melihat index
```
GET _cat/indices
```
- Melihat shards
```
GET _cat/shards?v
```

- Add new document
```
PUT belajar_elasticsearch/_doc/<id>
{
  "first_name": "Nauval",
  "last_name": "Shidqi",
  "address": {
    "country": "Indonesia",
    "city": "Jakarta"
  }, 
  "hobby": ["game", "coding", "travelling"]
}
```

- Bulk insert with json file
```
curl -H "Content-Type: application/x-ndjson" -XPOST http://localhost:9200/products/_bulk --data-binary "@products-bulk.json"
```

- Get document
```
GET belajar_elasticsearch/_doc/<id>
```

Dokumentasi CRUD: https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html

- Partial Update
```
POST belajar_elasticsearch/_update/<id>
{
  "doc": {
    "last_name": "Haruno"
  }
}
```

Delete field from elasticsearch document using painless scripting
```
POST belajar_elasticsearch/_update/2
{
    "script" : "ctx._source.remove(\"address.city\")"
}
```

- Searching
```
GET belajar_elasticsearch/_search
{
  "query": {
    "match": {
      "address.city": "malang"
    }
  }
}
```

- Searching tipe data array
```
GET belajar_elasticsearch/_search
{
  "query": {
    "terms": {
      "hobby": ["coding"]
    }
  }
}
```
match khusus spesifik 1 element pada array
```
GET belajar_elasticsearch/_search
{
  "query": {
    "match": {
      "hobby": "coding"
    }
  }
}
```

- Searching bool
```
GET belajar_elasticsearch/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "match": {
            "address.city": "malang"
          }
        }
      ], 
      "should": [
        {
          "match": {
            "first_name": "safira"
          }
        },
        {
          "match": {
            "last_name": "shidqi"
          }
        },
        {
          "term": {
            "hobby": "coding"
          }
        },
        {
          "term": {
            "hobby": "travelling"
          }
        }
      ]
    }
  }
}
```
Dokumentasi searching: https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html


- Aggregation (Terms Aggregation)
```
GET belajar_elasticsearch/_search
{
  "query": {
     "match_all": {}
  },
  "aggs": {
    "hobbies": {
      "terms": {
        "field": "hobby.keyword"
      }
    },
    "cities": {
      "terms": {
        "field": "address.city.keyword",
        "size": 10
      }
    }
  }
}
```
Contoh response
```
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 3,
    "successful" : 3,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "belajar_elasticsearch",
        "_type" : "_doc",
        "_id" : "abc",
        "_score" : 1.0,
        "_source" : {
          "first_name" : "Sakura",
          "last_name" : "Asa",
          "address" : {
            "country" : "Indonesia",
            "city" : "Jakarta"
          },
          "hobby" : [
            "travelling"
          ]
        }
      },
      {
        "_index" : "belajar_elasticsearch",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "first_name" : "Safira",
          "last_name" : "Shidqi",
          "address" : {
            "country" : "Indonesia",
            "city" : "Malang"
          },
          "hobby" : [
            "game",
            "coding",
            "travelling"
          ]
        }
      },
      {
        "_index" : "belajar_elasticsearch",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "first_name" : "Nauval",
          "last_name" : "Shidqi",
          "address" : {
            "country" : "Indonesia",
            "city" : "Jakarta"
          },
          "hobby" : [
            "game",
            "coding",
            "travelling"
          ]
        }
      }
    ]
  },
  "aggregations" : {
    "cities" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "Jakarta",
          "doc_count" : 2
        },
        {
          "key" : "Malang",
          "doc_count" : 1
        }
      ]
    },
    "hobbies" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "travelling",
          "doc_count" : 3
        },
        {
          "key" : "coding",
          "doc_count" : 2
        },
        {
          "key" : "game",
          "doc_count" : 2
        }
      ]
    }
  }
}
```
Dokumentasi aggregation: https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-terms-aggregation.html

- Wildcard (sama seperti LIKE / ILIKE pada SQL)
```
GET belajar_elasticsearch/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "wildcard": {
            "first_name": "*haru*"
          }
        }
      ]
    }
  }
}
```

## Mapping

- Membuat mapping
```
PUT /reviews
{
  "mappings": {
    "properties": {
      "rating": {"type": "float"},
      "content": {"type": "text"}, 
      "product_id": {"type": "integer"},
      "author": {
        "properties": {
          "first_name": {"type": "text"},
          "last_name": {"type": "text"},
          "email": {"type": "keyword"}
        }
      }
    }
  }
}

GET reviews/_mapping
```