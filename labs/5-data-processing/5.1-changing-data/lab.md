### Processors

- Can be used to transform documents before being index or reindexed into Elasticsearch.
- There are different ways to deploy preprocessors:
  - Elastic Agent
  - Logstash
  - Ingest Node Pipelines

- Elastic Agent processirs, Logstash filters, and ingest pipelines all have their own set of processors.
  - Several commonly used processors are in all three tools:

| Manipulate fields     | Manipulate values     | Special operations |
|-----------------------|-----------------------|--------------------|
| set                   | split/join            | csv/json           |
| remove                | grok                  | geoip              |
| rename                | dissect               | user_agent         |
| dot_expander          | gsub                  | script             |
| ...                   | ...                   | pipeline           |

### Ingest Node Pipelines

- Perform common transformations on your data before indexing it.
- Consist of a series of processors running serially.
- Are executed on ingest nodes

- Use Kibana Ingest Pipelines UI to create and manage pipelines.
  - View a list of your pipelines and drill down into details
  - Edit or clone existing pipelines
  - Delete pipelines

### Create the pipeline

... from Kibana UI

### Test the pipeline

... from Kibana UI

### Use the pipeline

- Apply the pipeline to documents in indexing requests

```http
POST new-index/_doc?pipeline=set_views
{
  "foo": "bar",
}
```

- Set a default pipeline for an index (only used if no other pipeline is used)

```http
PUT new-index
{
  "settings": {
    "default_pipeline": "set_views"
  }
}
```

### Set a final pipeline

```http
PUT new_index
{
  "settings": {
    "final_pipeline": "set_views"
  }
}
```

### Dissect Processor

- The dissect processor extracts structured fields out of a single text field within a document.

`1.2.3.4 [30/Apr/1998:22:00:52 +0000] \"GET /english/images/montpellier/18.gif\"`

`%{clientip} [%{timestamp}] \"%{verb} %{request}\"`

```json
"_source": {
  "request": "/english/images/montpellier/18.gif",
  "verb": "GET",
  "@timestamp": "30/Apr/1998:22:00:52 +0000",
  "clientip": "1.2.3.4"
}
```

### Pipeline Proecessor

- Create a pipeline that references other pipelines
  - Can be used with conditional statements

```http
PUT _ingest/pipeline/blogs_pipeline
{
  "processors": [
    {
      "pipeline": {
        "name": "inner_pipeline"
      }
    },
    {
      "set": {
        "field": "outer_pipeline_set",
        "value": "outer_value"
      }
    }
  ]
}
```

### Changing Data

- You can modifiy the `_source` using various Elasticsearch APIs:
  - _reindex
  - _update_by_query
- You have already seen some of these in previous labs.

### The Reindex API

- Source documents into a destination index
  - Source and destination indices must be different
- To reindex only a subset of the source index:
  - use `max_docs`
  - add a `query`

```http
POST _reindex
{
  "max_docs": 100,
  "source": {
    "index": "blogs",
    "query": {
      "match": {
        "versions": "6"
      }
    }
  },
  "dest": {
    "index": "blogs_fixed"
  }
}
```

### Apply a pipeline

- All documents from `old_index` will go trough the `set_views` pipeline before being indexed into `new_index`

```http
POST _reindex
{
  "source": {
    "index": "old_index"
  },
  "dest": {
    "index": "new_index",
    "pipeline": "set_views"
  }
}
```

### Reindex from a Remoto Cluster

- Connect to the remote Elasticsearch node using basic auth or API key
- Remote hsots have to be explicitly allowed in the `elasticsearch.yml` configuration file using the `reindex.remote.whitelist` property.

```http
POST _reindex
{
  "source": {
    "remote": {
      "host": "http://otherhost:9200",
      "username": "user",
      "password": "pass"
    },
    "index": "remote_index"
  },
  "dest": {
    "index": "local_index"
  }
}
```

### Udpate By Query API

- To change all the documents in an existing index use the Update By Query API
  - Reindexes every document into the same index
  - Update by query has many of the same features as reindex
  - Use a pipeline to update the _source

```http
POST blogs/_update_by_query?pipeline=set_views
{
  "query": {
    "match": {
      "category": "customers"
    }
  }
}
```

### The delete By Query API

- Use the Delete By Query API to delete documents that match a query
  - Deletes every document in the index that is a hit for the query

```http
POST blogs/_delete_by_query
{
  "query": {
    "match": {
      "author.title.keyword": "David Kravets"
    }
  }
}
```
