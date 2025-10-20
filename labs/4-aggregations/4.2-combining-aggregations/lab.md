### Working with Aggregations

- Combining aggregations
  - Specify different aggregations in a single request
  - Extract multiple insights from your data
- Change the aggregation's scope
  - Uses queries to limit documents on which an aggregation runs
  - Focus on specific, or relevant data
- Nest aggregations
  - Create a hierarchy of aggregations levels, or sub-aggregations, by nesting bucket aggregations within bucket aggregations
  - Use metric aggregations to calculate values over fields at any sub-aggregation lever in the hierarchy

### Reducing the Scope of an Aggregation

- By default, aggregations are performed on all documents in the index
- Combine with a query to reduce the scope

```http
GET blogs/_search?size=0
{
  "query": {
    "match": {
      "locale": "fr-fr"
    }
  },
  "aggs": {
    "no_of_authors": {
      "cardinality": {
        "field": "author.last_name.keyword"
      }
    }
  }
}
```

### Run Multiple Aggregations

- You can specify multiple aggregations in a single request

```http
GET blogs/_search?size=0
{
  "aggs": {
    "no_of_authors": {
      "cardinality": {
        "field": "author.last_name.keyword"
      }
    },
    "first_name_stats": {
      "string_stats": {
        "field": "author.first_name.keyword"
      }
    }
  }
}
```

### Sub-Aggregations

- Embed aggregations (bucket and metriic) inside other aggregations
  - Separate groups based on criteria
  - Apply metrics at vatious levels of the hierarchy
- No depth limit on nesting
- Bucket aggregations support bucket or metric sub-aggregations

```http
GET blogs/_search?size=0
{
  "aggs": {
    "by_month": {
      "date_histogram": {
        "field": "publish_date",
        "calendar_interval": "month"
      },
      "aggs": {
        "no_of_authors": {
          "cardinality": {
            "field": "author.last_name.keyword"
          }
        }
      }
    }
  }
}
```

### What is the sum of bytes per day?

```http
GET kibana_sample_data_logs/_search?size=0
{
  "aggs": {
    "requests_per_day": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "day"
      },
      "aggs": { 
        "daily_number_of_bytes": {
          "sum": {
            "field": "bytes"
          }
        },
        "median_number_of_bytes": {
          "percentiles": {
            "field": "bytes",
            "percents": [50]
          } 
        }
      }
    }
  }
}
```

### On what day were the most bytes served?

```http
GET kibana_sample_data_logs/_search?size=0
{
  "aggs": {
    "requests_per_day": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "day",
        "order": { "daily_number_of_bytes": "desc" }
      },
      "aggs": { 
        "daily_number_of_bytes": {
          "sum": {
            "field": "bytes"
          }
        },
        "median_number_of_bytes": {
          "percentiles": {
            "field": "bytes",
            "percents": [50]
          } 
        }
      }
    }
  }
}
```

### What are the most popular operating systems per month?

```http
GET kibana_sample_data_logs/_search?size=0
{
  "aggs": {
    "logs_by_month": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "month"
      },
      "aggs": {
        "machine_os": {
          "terms": {
            "field": "machine.os.keyword",
            "size": 10
          }
        }
      }
    }
  }
}
```

### Pipeline Aggregations

- Work on output produced from other aggregations
- Examples
  - bucket min/max/sum/avg
  - cumulative_sum
  - moving_avg
  - bucket_sort

```http
"aggs": {
  "blogs_by_month": {
    "date_histogram": {
      "field": "publish_date",
      "calendar_interval": "month"
    },
    "aggs": {
      "no_of_authors": {
        "cardinality": {
          "field": "author.last_name.keyword"
        }
      },
      "diff_author_ct": {
        "derivative": {
          "buckets_path": "no_of_authors"
        }
      }
    }
  }
}