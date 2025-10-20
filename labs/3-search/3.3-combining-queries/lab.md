### Combining Queries using Boolean Logic

- Suppose you want to write the following search query:
  - Find blogs about "agent" written in English
- This search is actually a combination of two queries:
  - "agent" neeeds too be in the content or title field
  - And "en-us" needs to be in the locale field
- How can you combine these two queries?
  - By using Boolean logic and the `bool` query

### The Bool Query

- The `bool` query combines one or more boolean clauses
  - must
  - filter
  - must_not
  - should
- Each of the clauses is optional
- Clauses can be combined
- Any clause accepts oone or more queries

```http
GET blogs/_search
{
  "query": {
    "bool": {
      "must": [],
      "filter": [],
      "must_not": [],
      "should": []
    }
  }
}
```

### The Must Clause

- Any query in the `must` clause **MUST** match for a document to be a hit
- Every query contributes to the `_score` of the document

```http
GET blogs/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "content": "agent"
          }
        },
        {
          "match": {
            "locale": "en-us"
          }
        }
      ]
    }
  }
}
```

### The Filter Clause

- Filters are like `must` clauses:: any quert in the `filter` clause **MUST** match for a document to be a hit
- But, queries in the `filter` clause do not contribute to the `_score` of the document

```http
GET blogs/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "content": "agent"
          }
        },
      ],
      "filter": [
        {
          "match": {
            "locale": "en-us"
          }
        }
      ]
    }
  }
}
```

### The Must Not Clause

- Use the `must_not` clause to exclude documents that match the specified queries
- Queries in the `must_not` clause do not contribute to the `_score` of the document

```http
GET blogs/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "content": "agent"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "locale": "en-us"
          }
        }
      ]
    }
  }
}
```

### The Should Clause

- Use the `should` to boost documents that match a query
  - queries in the `should` clause contribute to the `_score` of the document
  - documents that do not match the queries in a `should` are returned as hits anyway
- Use the `minimum_should_match` parameter to specify the number or the percentage of `should` clauses that must match for a document to be a hit

```http
GET blogs/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "content": "agent"
          }
        }
      ],
      "should": [
        {
          "match": {
            "locale": "en-us"
          }
        },
        {
          "match": {
            "locale": "fr-fr"
          }
        }
      ],
      "minimum_should_match": 1
    }
  }
}
```

Uses filter as much as possible for better performance!

### Comparing Quert and Filter Contexts

| Query Context                         | Filter Context                                |
|---------------------------------------|-----------------------------------------------|
| `must`                                | `filter`                                      |
| `should`                              | `must_not`                                    |
| Calculates a scoore                   | Skips score calculation                       |
| Slower                                | Faster                                        |
| No automatic caching                  | Automatic caching for frequently used filters |

### The bool Query Summary

| Clause   | Exclude docs | Scoring |
|----------|--------------|---------|
| must     | v            | v       |
| must_not | v            | x       |
| should   | x'           | v       |
| filter   | v            | x       |

> x': If the bool query includes at least one should clause, and no must or filter clauses, it will include documents that match
and exlude documents that don't