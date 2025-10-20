### Transformm Your Data for Better Insights

- Summarize existing Elasticsearch indices using aggregations to create more efficient datasets
  - **pivot** event-centric data into entity-centric indices for improved analysis
  - retrive the **latest** document based on a unique keym simplifying time series data

### Cluster-efficient Aggregations

- Elasticsearch Aggregations provide powerful insights but can be resource-intensive with large datasets
  - Complex aggregations on large volumes of data may lead to memory issues or performance bottlenecks
- Common challenges:
  - Need for a complete feature index (vs. just top N items)
  - Need to sort aggregation results using pipeline aggregations
  - Want to create summary tables to optimize query performance
- Solution:
  - **Transform your data** to create more efficient and scalable summaries for faster, optimized querying

### Example

- The **blogs** index is a collection of all the blogs
- The **web_traffic** index is a record of visitors to our blogs
- How many visitors visited each blog?

### Transform web_traffic

- Transform the **web_traffic** using **Pivot**
  - Group by url
- Choose aggregations of stats you want to collect for each url
  - count of docs = number of visit
  - avg (runtime_sec) = average time to load page
- You can also edit runtime mappings directly in the UI

### Configuring Transform Settings

- Continuous mode
  - transforms run continuously, processing new data as it arrives
- Retention policy
  - identify and manage out of date documents in the destination index
- Checkpoints
  - created each time new source data is ingested and transformed
- Frequency
  - advanced option to set interrval between checkpoints (max 1 hour)

### Destination Index

- Pre-create the destination index with custom settings for performance
  - use the **Preview transform API** to review **generated_dest_index**
  - optimize index mappings and settings for efficient storage and qquerying
  - disable _source to reduce storage usage
  - use index sorting if grouping by multiple fields

### Latest Transform

- Use **Latest** transforms to copy the most recent documents into a new index
- Examples
  - track the latest purchase for each customer
  - capture the latest event for each host