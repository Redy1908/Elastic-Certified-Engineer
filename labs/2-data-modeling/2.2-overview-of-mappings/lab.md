### What is a Mapping?

- A mapping is a **pre-index** schema definition that contains:
  - Field names
  - Data types for each field
  - How the fields should be indexed and stored
- Elastichsearch will happily index any document without knowing its details (number of filds, thei data types, etc)
  - However, behind the scenes, Elasticsearch assigns **data types** to your fields in a mapping

### Elasticsearch Data Types for Fields

- Simple types:
  - `text`: Full-text search fields
  - `keyword`: Exact value fields
  - `date` and `date_nanos`: Date fields with different precisions
  - `long`, `integer`, `short`, `byte`: Numeric fields
  - `double`, `float`, `half_float`, `scaled_float`: Floating point numeric fields
  - `boolean`: True/false fields
  - `binary`: Binary data fields
  - `geo`: Geospatial data fields
- Hierarchical types: Like `object` and `nested`

For a full list of data types, see the [Elasticsearch documentation on data types](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html).

### Defining a Mapping

- In many use cases, you will need to define your own mapping
- Defined in the `mappings` section when creating an index:

```http
PUT /my_index
{
  "mappings": {
    define mappings here
  }
}
```

Or added to an existing index:

```http
PUT /my_index/_mapping
{
  define mappings here
}
```

### Why have we not defined a mapping yet?

When you index a document with unmapped fields, Elasticsearch will dynamically create the mapping for those fields
  - Fields not already defined in a mapping are added

What will be the mapping for the following document?

```http
POST /my_index/_doc
{
  "name": "John Doe",
  "age": 30,
  "is_employee": true,
  "join_date": "2023-01-15"
}
```

```http
GET /my_index/_mapping
```

```json
{
  "my_index": {
    "mappings": {
      "properties": {
        "age": {
          "type": "long"
        },
        "is_employee": {
          "type": "boolean"
        },
        "join_date": {
          "type": "date"
        },
        "name": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        }
      }
    }
  }
}
```

### Multi-Field Mapping

Elasticsearch has no way of knowing if a field should be treated as both `text` and `keyword` unless you tell it to do so
  - This is where **multi-field** mappings come in
- A multi-field mapping allows the same field to be indexed in different ways for different purposes
  - For example, the `name` field above is indexed as both `text` (for full-text search) and `keyword` (for exact matches and aggregations)

### Multi-Field in the Mapping

```json
{
  "my_index": {
    "mappings": {
      "properties": {
        "name": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        }
      }
    }
  }
}
```

- The `name` field is indexed as `text` and `keyword` by default
- Elasticsearch automatically creates the `name.keyword` sub-field
- The name of a multi-field (in this case, `keyword`) can be customized as needed

### Do you always need text and keyword?

- Probably not! Indexing every string twice:
  - **slows** down indexing performance
  - **increases** storage requirements
- Think what do you want to do with eahch string field:
  - Full-text search? Use `text`
  - Case-insensitive? Use `text`
  - Exact term matches? Use `keyword`
  - Aggregations & sorting? Use `keyword`
- Optimize the mapping based on your use case!

### Dynamic Mapping is Rarely Ideal

- For example, the default for an integer is `long`
  - Not always appropriate for the content
- A more tailored type can help save on memory and speed

### Can you change a mapping after the index is created?

- **No** - not without **reindexing** your documents
  - Adding new fields is allowed
  - All other mapping changes require reindexing
- Why not?
  - If you could switch a field's data type, all the values that were already indexed before the switch would become unsearchable on that field
- Invest time upfront to design the mapping carefully!

### Fixing a Mapping

- Create a new index with the correct mapping

```http
PUT /my_index_v2
{
  "mappings": {
    define correct mappings here
  }
}
```

### The Reindex API

- To populate the new index, use the `_reindex` API

```http
POST /_reindex
{
  "source": {
    "index": "my_index"
  },
  "dest": {
    "index": "my_index_v2"
  }
}
```

### Definig your Own Mapping

- Kibana [file uploader](https://127.0.0.1:5601/app/home#/tutorial_directory/fileDataViz) does an excellent job of guessing data types
  - Allows you to customize the mapping before index creation

### Defining your Own Mapping Manually

- If not using the file uploader, to define an explici mapping:
  - Index a sample document that contains the fileds you want defined in the mapping into a dummmy index
  - Get the dynamic mapping created by Elasticsearch
  - Modify the mapping as needed
  - Create the final index with the modified mapping

### Step 1: Index a Sample Document into a Dummy Index

- Use values that will map closely to the data types you want

```http
PUT /dummy_index/_doc/1
{
  "name": "Alice Smith",
  "age": 28,
  "is_employee": false,
  "join_date": "2022-05-20",
  "salary": 55000.75
}
```

### Step 2: Get the Dynamic Mapping Created by Elasticsearch

- GET the mapping, then copy-paste it into the [Console](https://127.0.0.1:5601/app/dev_tools#/console/shell)
  - In Kibana's file uploader, this is in the Advanced section after Import

```http
GET /dummy_index/_mapping
```

```json
{
  "dummy_index": {
    "mappings": {
      "properties": {
        "age": {
          "type": "long"
        },
        "is_employee": {
          "type": "boolean"
        },
        "join_date": {
          "type": "date"
        },
        "name": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "salary": {
          "type": "float"
        }
      }
    }
  }
}
```

### Step 3: Modify the Mapping as Needed

- We change the `age` field from `long` to `integer`

```json
{
  "mappings": {
    "properties": {
      "age": {
        "type": "integer"
      },
      "is_employee": {
        "type": "boolean"
      },
      "join_date": {
        "type": "date"
      },
      "name": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "salary": {
        "type": "float"
      }
    }
  }
}
```

### Step 4: Create the Final Index with the Modified Mapping

```http
PUT /final_index
{
  "mappings": {
    "properties": {
      "age": {
        "type": "integer"
      },
      "is_employee": {
        "type": "boolean"
      },
      "join_date": {
        "type": "date"
      },
      "name": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "salary": {
        "type": "float"
      }
    }
  }
}
```

Great Mapping Means Great Performance!

- Fast indexing
- Optimized storage
- Accurate search results