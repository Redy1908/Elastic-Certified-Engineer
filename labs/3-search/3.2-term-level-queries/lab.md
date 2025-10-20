### Matching Exact Terms

- Recall that `full text` queries are analyzed and then searched within the index.
- `Term-level queries` are used for exact searches
  - Do not analyze the search terms
  - Returns the exact match of the orginal string as it occurs in the documents

### Term Level Queries

- Find documents based on precise values in structured data
  - Queries are matched on the exact terms store in a field

| Full text queries | Term level queries| Many more      |
|-------------------|-------------------|----------------|
| match             | term              | script         |
| match_phrase      | range             | percolate      |
| multi_match       | exists            | span_queries   |
| query_string      | fuzzy             | geo_queries    |
| ...               | regexp            | nested         |
|                   | wildcard          | ...            |
|                   | ids               |                |
|                   | ...               |                |

### Matching on a Keyword Field

- Use the `keyword` field to match on an **exact term**
  - The `term` must exactly match the field value, including whitespace and capitalization
- Recall that keyword types are commonly used for:
  - Structured content such as IDs, email, hostnames, or zip codes
  - Soorting and aggregations

```http
GET blogs/_search
{
  "query": {
    "term": {
      "authors.job_title.keyword": "Senior Software Engineer"
    }
  }
}
```

### Searching for Numbers, Dates, and IPs

- So far we have used the full-text queries
  - Great for searching words in bodies of text
- How would we query for dates, numbers, or IP addresses?
  - For example:
    - find blogs created after a certain date
    - find blogs with more than a certain number of views
  - Use the `range` query

### The Range Query

- Use the following parameters to specify a range:
  - `gt` (greater than)
  - `gte` (greater than or equal to)
  - `lt` (less than)
  - `lte` (less than or equal to)
- Ranges can be open-ended

```http
GET blogs/_search
{
  "query": {
    "range": {
      "publish_date": {
        "gte": "2023-01-01",
        "lt": "2024-01-01"
      }
    }
  }
}
```

### Date Math

- Use date math to express relatiuve dates in range queries

| y      | years   |
|--------|-------- |
| M      | months  |
| w      | weeks   |
| d      | days    |
| h or H | hours   |
| m      | minutes |
| s      | seconds |

Examples:

- now-1h
- now+1h+30m
- now/d+1d
- 2024-01-15||+1M

```http
GET blogs/_search
{
  "query": {
    "range": {
      "publish_date": {
        "gte": "now-30d/d"
      }
    }
  }
}
```

### The Exists Query

- Returns documents that contain an indexed value for a field
- Empty strings also indicate that a field exists

```http
GET blogs/_count
{
  "query": {
    "exists": {
      "field": "category"
    }
  }
}
```

### Async Search

- Search asynchronously
- Useful foor slow queries and aggregations
  - monitor the progress
  - retrive partial results as they become available

```http
POST blogs/_async_search?wait_for_completion_timeout=0s
{
  "query": {
    "match": {
      "title": "community team"
    }
  }
}
```

- Response:

```json
{
  "id": "some-unique-id",
  "is_partial": true,
  "is_running": true,
  "start_time_in_millis": 1700000000000,
  "expiration_time_in_millis": 1700000060000
  ...
}
```

- The `id` can be used to retrive the results later (`GET /_async_search/{id}`)
- `is_partial` indicates whether the current set of results is partial
- `is_running` indicates whether the query is still running
- You can retrive the results until `expiration_time_in_millis` (defaults to 5 days)
