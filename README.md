![elasticsearch-azure](https://user-images.githubusercontent.com/38520491/129489539-cff190f0-08c2-46ce-ae32-c4e22e599b9a.png)

### Cluster health
```
GET _cluster/health/
```
### Show index mapping
```
GET /{index-name}/_mapping
```
### Create snapshots
```
PUT /_snapshot/{repository_name}/{snapshot-indexname-2020}
{
 "indices": "index-2020.09,index-2020.10,index-2020.11,index-2020.12",
 "ignore_unavailable": false,
 "include_global_state": false,
 "metadata": {
 "taken_by": "admin",
 "taken_because": "index backup"
  }
}
```
### Show snapshot information
```
GET /_snapshot/_all
```
### Show taken snapshots in specific repository
```
GET /_snapshot/{repository-name}/_all
```
### Show global cluster setting
```
GET /_cluster/settings/
```
### Change some cluster setting
```
PUT _cluster/settings
{
  "transient": {
    "search.max_buckets": 10000000
  }
}
```
### Show running tasks
```
GET /_cat/tasks?v
```
### Kill specific task
```
POST _tasks/{task:ID}/_cancel
```
### Show index information
```
GET /{index-name}/_stats
```
### Change some index setting
```
PUT /{index-name}-*/_settings
{
  "index.allocation.max_retries": "10"
}
```
### The size of the cache (in bytes) and the number of evictions per indices.
```
GET /_stats/request_cache?human
```
### The size of the cache (in bytes) and the number of evictions per nodes
```
GET /_nodes/stats/indices/request_cache?human
```
### Show statistics for one or more indices.
```
GET /{index-name}/_stats
```
### Forces a merge on the shards of one or more indices. with `only_expunge_deletes=true` means only expunge segments containing document deletions, default is false.
```
POST /{index-name}/_forcemerge?only_expunge_deletes=true
```
### Update indices with inline script. For example delete some field
```
POST {index-name}/_update_by_query?conflicts=proceed&refresh
{
 "script": {
    "inline": "ctx._source.remove(\"some-field-name\")"
  }
}
```
### Update indices with must statement, 
```
POST /{index-name}/_update_by_query
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "tags.keyword": "some-tag-must-exists"
          }
        }
      ]
    }
},
    "script": {
    "inline": "ctx._source.tags = 'some-tag-to-insert'",
    "lang": "painless"
  }
}
```
### Delete some documents with must statement in field
```
POST /{index-name}/_delete_by_query?conflicts=proceed
{
  "query": {
    "match": {
      "client_ip": {  #Or any field name
        "query": "x.x.x.x"
      }
    }
  }
}
```
### Delete some documents with must statement in field
```
POST {index-name}/_delete_by_query
{
  "query": {
    "bool": {
      "must": [],
      "filter": [
        {
          "match_all": {}
        },
        {
          "match_phrase": {
            "any_field.keyword": {
              "query": "foobar"
            }
          }
        }
      ]
    }
  }
}
```
### Delete some documents in specific time rang and must statement in field
```
POST {index-name}/_delete_by_query?conflicts=proceed&refresh
{
  "query": {
    "bool": {
      "must": [],
      "filter": [
        {
          "match_all": {}
        },
        {
          "match_phrase": {
            "agent.hostname.keyword": {  #Or any field name
              "query": "Agent_Name"
            }
          }
        },
        {
          "range": {
            "@timestamp": {
              "format": "strict_date_optional_time",
              "gte": "2020-12-18T20:30:00.000Z",
              "lte": "2020-12-19T20:30:00.000Z"
            }
          }
        }
      ]
    }
  }
}
```
### Delete some documents with must and must not staement
```
POST {index-name}/_delete_by_query
{
    "query": {
    "bool": {
      "must": [],
      "filter": [
        {
          "match_all": {}
        },
        {
          "match_phrase": {
            "tags.keyword": {  #Or any field name
              "query": "foobar"
            }
          }
        }
      ],
      "should": [],
      "must_not": [
        {
          "match_phrase": {
            "app.keyword": {  #Or any field name that should not exists
              "query": "somefoobar"
            }
          }
        }
      ]
    }
  }
}
```
### Delete some documents with must and must not and time rang 
```
POST {index-name}/_delete_by_query?conflicts=proceed
{
  "query": {
    "bool": {
      "must": [],
      "filter": [
        {
          "match_all": {}
        },
        {
          "match_phrase": {
            "agent.hostname.keyword": {  #Or any field name
              "query": "foobar"
            }
          }
        },
        {
          "range": {
            "@timestamp": {
              "format": "strict_date_optional_time",
             "gte": "2020-10-27T20:24:56.182Z",
              "lte": "2020-11-01T23:35:21.353Z"
            }
          }
        }
      ],
      "should": [],
      "must_not": [
        {
          "match_phrase": {
            "app.keyword": {  #Or any field name that should not exists
              "query": "foobar" 
            }
          }
        }
      ]
    }
  }
}
```
### Remove field from documents based on time range
```
POST {index-name}/_update_by_query?requests_per_second=-1
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "@timestamp": {
              "format": "strict_date_optional_time",
              "gte": "2020-12-13T20:30:00.000Z",
              "lte": "2020-12-20T20:30:00.000Z"
            }
          }
        }
      ]
    }
},
 "script": {
    "source": "ctx._source.remove(\"field-name\")"
  }
}
```
### Delete documents with must and must not in field
```
POST {index-name}/_delete_by_query?conflicts=proceed&refresh
{
  "query": {
    "bool": {
      "must": [],
      "filter": [
        {
          "match_all": {}
        },
        {
          "exists": {
            "field": "store" #field must exists
          }
        }
      ],
      "should": [],
      "must_not": [
        {
          "bool": {
            "should": [
              {
                "match_phrase": {
                  "store.keyword": "8" #must not be this value
                }
              },
              {
                "match_phrase": {
                  "store.keyword": "32" #must not be this value
                }
              }
            ]
          }
        }
      ]
    }
  }
}
```
### Remove field if another field exists
```
POST {index-name}/_update_by_query
{
  "script": "ctx._source.remove('some-field-to-remove')",
  "query": {
    "bool": {
      "must": [
        {
          "exists": {
            "field": "some-field-should-exists"
          }
        }
      ]
    }
  }
}
```
### Remove field if another field match something
```
POST {index-name}/_update_by_query?conflicts=proceed&refresh
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "tags.keyword": "foobar" #Or any field name
          }
        }
      ]
    }
},
 "script": {
    "inline": "ctx._source.remove(\"field-name-to-remove\")"
  }
}
```
### Reindex some documents with must
```
POST /_reindex
{
   "source": {
    "index": "source-index-name",
    "query": {
    "bool": {
      "must": [],
      "filter": [
        {
          "match_all": {}
        },
        {
          "match_phrase": {
            "app.keyword": { #Or any field name and has this value
              "query": "foobar"
            }
          }
        },
        {
          "match_phrase": {
            "_index": {
              "query": "source-index-name"
            }
          }
        }
      ]
    }
  }
},
  "dest": {
    "index": "destination-index-name"
  }
}
```
