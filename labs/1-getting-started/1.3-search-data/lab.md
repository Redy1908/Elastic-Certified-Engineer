### Query Languages

- Used in Kibana query bar (find it at the top of [Discover page](https://127.0.0.1:5601/app/discover)):
  - KQL
  - Lucene
  - ES|QL
- Query DSL: Gives access to all Elasticsearch features (Most flexible) <- Our focus
- Elasticsearch SQL: Search indices using familiar SQL syntax
- EQL: For security data, threat hunting

### Basic Structure of Search

In Elasticsearch, search breaks down into two basic parts:
- Query:
  - Which documents meet a specific set of criteria?
- Aggregations:
  - Tell me something about a group of documents

### Using Query DSL

Send a request using the search API:
- **GET** <index>/_search

```http
GET /my-index/_search
{
  "query": {
    "match_all": {}
  }
}
```

The `match_all` query is the simplest and default query for the search API (we can omit it entirely).
- Every document is a hit for this search
- Elasticsearch returns the first 10 hits by default

```http
GET /my-index/_search
```

We can replace `my-index` with a comma separated list or use the wildcard `*` to search all indices.

### Aggregations

Visualizations in Kibana are powered by aggregations.

Differences between queries and aggregations:
- Queries: Elasticsearch returns documents that match the search criteria
- Aggregations: Once the correct documents are found, aggregations summarize or compute metrics on those documents

### Let's Aggregate data

Suppose you  want to know what date the first blog was published. You can use an aggregation to find out.

```http
GET /blogs/_search
{
  "aggs": {
    "first_blog": {
      "min": {
        "field": "publish_date"
      }
    }
  }
}
```

### Elasticsearch Query Language (ES|QL)

- A piped query language that delivers advanced search capabilities
  - Streammlines searching, aggregating, and visualizing large data sets
  - Brings toghether capabilities of multiple query languages (Query DSL, KQL, EQL, Lucene, SQL...)
- Powered by a dedicated query engine with concurrency processing
  - Designed for performance
  - Enhances speed and efficiency irrespective of data source or structure

An ES|QL query example is composed of a series of commands chained together using the pipe (`|`) operator.

```
source-command | processor-command1 | processor-command2 | ...
```

- Source Command: Retrives or generate data in the form of tables
- Processing Commands: Take a table as input and produces a table as output

### Running an ES|QL Query in Dev Tools

Wrap the query in a POST request to the `_query` endpoint.
- By default, results are returned as JSON object
- Use the `format` option to retrive the results in alternative formats (json, tsv, txt, yaml, cbor and smile)

```http
POST /_query
{
  "query": "FROM blogs | KEEP publish_date, author.first_name | SORT (publish_date)"
}
```

```http
POST /_query?format=csv
{
  "query": """
    FROM blogs
    | KEEP publish_date, author.first_name, author.last_name
    | SORT (publish_date)
  """
}
```

### ES|QL Examples

- List of columns to KEEP
  ```http
  FROM blogs
  | KEEP publish_date, author.first_name, author.last_name
  ```
- Filter results **WHERE** the autor's last name is "Kearns"
  ```http
  FROM blogs
  | WHERE author.last_name == "Kearns"
  | KEEP publish_date, author.first_name, author.last_name
  ```
- **STATS...BY** groups rows to calculate aggregated values
  ```http
  FROM blogs
  | STATS count = COUNT(*) BY author.last_name.keyword
  | SORT count DESC
  | LIMIT 10
  ```
