### Query DSL Overview

A search language for Elasticsearch

- Query
- Aggregate
- Sort
- Filter
- Manipulate responses

```http

GET blogs/_search
{
  "query": {
    "match": {
      "title": "community team"
    }
  }
}
```

### The Match Query

- Returns documents that match a provided text, number, date or boolean value
- By default, the match query
  - uses the `OR` operator to combine individual term matches
  - is `case insensitive`

The `operator` parameter;

- defines the logic used to interpret text
- specify `or` (default) or `and`

```http
GET blogs/_search
{
  "query": {
    "match": {
      "title": {
        "query": "community team",
        "operator": "and"
      }
    }
  }
}
```

- The `or` or `and` options might be too wide or too strict
- Use the `minimum_should_match` parameter to specify a more flexible matching requirement
  - Specifies the minimum number of clauses that must match
  - Trims the long tail of less relevant results

```http
GET blogs/_search
{
  "query": {
    "match": {
      "title": {
        "query": "elastic community team",
        "minimum_should_match": "2"
      }
    }
  }
}
```

- The match query does not consider:
  - the order of the terms
  - how far apart the terms are (even if the `and` operator is used)

### The Match Phrase Query

- The `match_phrase` query searchs foor the exact sequence of terms specified in the query
  - Terms in the phrase must appear in the exact order 
  - Use the `slop` parameter to specify how far apart terms are allowed for it to be considered a match

```http
GET blogs/_search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "community team",
        "slop": 1
      }
    }
  }
}
```

### Searching Multiple Fields

- How would you query multiple fields at once?
  - For example:
    - find blogs that mention "Agent" in the title or content fields
- Use the `multi_match` query
  - specify the comma-delimited list of fields using square brackets `[]`

```http
GET blogs/_search
{
  "query": {
    "multi_match": {
      "query": "agent",
      "fields": ["title", "content"]
    }
  }
}
```

- By default, the best scoring field will determine the score
  - set the `type` parameter to `most_fields` to let the score be the sum of the scores of the individual fields:

  ```http
  GET blogs/_search
  {
    "query": {
      "multi_match": {
        "query": "elastic community",
        "fields": ["title", "content"],
        "type": "most_fields"
      }
    }
  }
  ```

- You can search for phrases with the multi_match query
  - set the `type` parameter to `phrase`

  ```http
  GET blogs/_search
  {
    "query": {
      "multi_match": {
        "query": "elastic community",
        "fields": ["title", "content"],
        "type": "phrase"
      }
    }
  }
  ```

  ### Score

  - Elasticsearch calculates a scoore for each document that is a hit
    - Ranks search result based on relevance
    - Represents how well a document matchs a given search query
  - BM25
    - Default scoring algorithm in Elasticsearch
    - Based on 
      - Term frequency (TF): The more a term appears in a field, the more important it is
      - Inverse document frequency (IDF): The more documents that contain the term, the less important it is
      - Field length: shorter fields are more likely to be relevant than longer fields

### Query Response

- By default the query response will return:
  - The top 10 documents that match the query
  - Scoored by `_score` in descending order

Explicit default query:

```http
GET blogs/_search
{
  "from": 0,
  "size": 10,
  "sort": {
    "_score": {
      "order": "desc"
    }
  },
  "query": {
     ...
  }
}
```

Example (if two documents have the same publish_date, sort them by `_score`):

```http
GET blogs/_search
{
  "from": 100,
  "size": 50,
  "sort": [
    {
      "publish_date": {
        "order": "asc"
      }
    },
    "_score"
  ],
  query": {
    ...
  }
}
```

### Sorting

- Use `keywords field` to sort on field values
- Results are `not scored`
  - `_score` has no impact on sorting
  - `_score`: null

```http
GET blogs/_search
{
  "query": {
    "match": {
      "title": "Elastic"
    }
  },
  "sort": {
    "title.keyword": {
      "order": "asc"
    }
  }
}
```

### Retrive Selected Fields

- By default, the entire `_source` document is returned in the search response
  - The original data that was indexed
- Use `fields` parameter to retrieve specific fields only

```http
GET blogs/_search
{
  "_source": false,
  "fields": ["title", "publish_date"],
  ...
}