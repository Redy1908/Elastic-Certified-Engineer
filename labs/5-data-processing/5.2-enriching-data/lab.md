### Enrichment Uses Cases

- Add zip codes based on geo location
- Entichment based on IP range
- Currenci conversion
- Denomalizing data
- Threat Intel Enrichment

### Denormalize your Data

- At its heart, Elasticsearch is a flat hierarchy and trying to force relationla data into it can be very challenging.
- Documents should be moodeled so that search time operations  are as cheap as possible.
- Denormalization gives you the most power and flexibility
  - Optimize for reads
  - No need to perform expensive joins

- Denormalizing your data refers to **flattening** your data
  - Storing reduntant copies of data in each document instead oof using some type of relationship
- There are various ways too denormalize your data
  - Outside Elasticsearch
    - Write your own application side join
    - Logstash filters (eg, JDBC driver)
  - Inside Elasticsearch
    - Enrich processor in ingest node pipelines

### Denormalize Blogs

- In the blogs index, there is a category field with ID values
- There is an associated categories index with uid and title fields

### The enrich processor

- Use the enrich processor to add data from your existing indices to incoming documents during ingest
- There are several steps to enriching your data
  1. Set up an enrich policy
  2. Create an enrich index for the policy
  3. Create an ingest pipeline with an enrich processor
  4. Use the pipeline

### Step 1: Set up an enrich policy

```
PUT _enrich/policy/cat_policy
{
  "match": {
    "indices": "categories",
    "match_field": "uid",
    "enrich_fields": [ "title" ]
  }
}
```

> Once created, you can't update or change an enrich policy.

### Step 2: Create an enrich index for the policy

- Execute the enrich policy to create the enrich index for your policy

`POST _enrich/policy/cat_policy/_execute`

- When executed, the enrich policy create a system index called the enrich index
  - The processor uses this index to match and enrich incoming documents
  - It is read only, meaning you can't directly change it
  - More efficient that directly matching incoming document to the source indices

### Step 3: Create an ingest pipeline with an enrich processor

```
PUT /_ingest/pipeline/categories_pipeline
{
  "processors": [
    {
      "enrich": {
        "policy_name": "cat_policy",
        "field": "category",  -> the field in the input document that matches the policy's match_field
        "target_field": "cat_title",
      }
    }
  ]
}
```

> Set max_matches > 1 if the field in the input document is an array.

### Step 4: Use the pipeline

- Finally updated each document with the enriched data

`POST /blogs/_update_by_query?pipeline=categories_pipeline`

> You can also set the pipeline as the default pipeline for an index to enrich incoming documents.

### Updating the policy

Once created, you can't update or change an enrich policy. Instead, you can:
  1. Create and execute a new enrich policy
  2. Replace the previous enrich policy with the new enrich policy in any in-use enrich processors
  3. Use the delete enrich policy API or Index Management in Kibana to delete the previous enrich policy

### Updating an enrich index

- Once created, you cannot update or index documents to an enrich index. Instead,
  - Update your source indices and execute the enrich policy again
  - This creates a new enrich index from your updated source indices
  - Previous enrich index will deleted with a delayed maintennece job, by default this is done every 15 minutes
- You can reindex or update any already ingested documents using your ingest pipeline

### Performance considerations

- The enrich processor performs several operations and may impact the speed of your ingest pipeline
- We strongly recomment testing and benchmarking your enrich processor before deploying to production
- We do not recoommend using the enrich processor to append real-time data. The enrich processor is best suited for data that changes infrequently