 ### Mapping Parameters

 - In addition to the type, fields in a mapping can be configured with additional parameters
   - For example to set the analyzer for a text field:
   
   ```json
   {
     "mappings": {
       "properties": {
         "title": {
           "type": "text",
           "analyzer": "english"
         }
       }
     }
   }
   ```
- Complete list of mapping parameters can be found in the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-params.html)

### Date Formats

- Use the `format` parameter to specify custom date formats for date fields
  - Defaults to ISO 8601 format
- Choose from built-in formats or define your own format

```json
{
  "mappings": {
    "properties": {
      "my_date_field": {
        "type": "date",
        "format": "dd/MM/yyyy HH:mm:ss||epoch_millis"
      }
    }
  }
}
```

### Coercing Data

- By default, Elasticsearch attempts to coerce data to match the data type of the field
  - For example, suppose the `ratig` field is a long:

```http
PUT ratings/_doc/1
{
  "ratig": "5"
}
```

```http
PUT ratings/_doc/2
{
  "ratig": 4.7
}
```

```http
PUT ratings/_doc/3
{
  "ratig": "5"
}
```

All three documents will be indexed successfully. Rating values will be coerced to a long data type.

### Coercing Data

You can disable coercion if you want Elasticsearch to reject documents that have unexpected values

```
{
  "mappings": {
    "properties": {
      "ratig": {
        "type": "long",
        "coerce": false
      }
    }
  }
}
```

### Not Storing Doc Values

- By default, Elasticsearch creates **doc values** data structure for many fileds during indexing
  - doc values enale you to aggregate/sort on those fields
  - but take up disk space
- Fields that won't be used for aggregations or sorting can be configured to not store doc values
  - set `doc_values` parameter to `false`

```json
{
  "mappings": {
    "properties": {
      "description": {
        "type": "text",
        "doc_values": false
      }
    }
  }
}
```

### Not Indexing a Field

- By default, for every field Elasticsearch creates a data structure that enables fast queries
  - **inverted index** (e.g., for `text`, `keyword`) or **BKD tree** (e.g., for `geo`, `numeric`)
  - Set **index** parameter to `false` to disable indexing for a field that do not require fast querying
    - Fields with doc values still support slower queries

```json
{
  "mappings": {
    "properties": {
      "notes": {
        "type": "text",
        "index": false
      }
    }
  }
}
```

### Disabling a Field

- A field that won't be used at all and should just be stored in `_source`
  - Set `enabled` parameter to `false`

```json
{
  "mappings": {
    "properties": {
      "metadata": {
        "type": "object",
        "enabled": false
      }
    }
  }
}
```

The field `metadata` can be retrived from `_source`, but it is not searchable or sotred in any other data structures.

### The copy_to Parameter

- Consider a document with three location fields:

```http
POST location/_doc
{
  "region_name": "Victoria",
  "country_name": "Australia",
  "city_name": "Surrey Hills"
}
```

- You could use a **bool/multi_match** query to search all three fields
- OOr you could copy all three values to a single field during indexing using the `copy_to` parameter

```json
{
  "mappings": {
    "properties": {
      "region_name": {
        "type": "text",
        "index": "false",
        "copy_to": "all_location"
      },
      "country_name": {
        "type": "text",
        "index": "false",
        "copy_to": "all_location"
      },
      "city_name": {
        "type": "text",
        "index": "false",
        "copy_to": "all_location"
      },
      "all_location": {
        "type": "text"
      }
    }
  }
}
```

- The `all_location` field is not stored in the `_source`
  - But it is indexed, so you can query on it

```http
GET location/_search
{
  "query": {
    "match": {
      "all_location": "victoria australia"
    }
  }
}
```

### Use Case for Dynamic Templates

- Manually defining a mapping can be tedious when you:
  - Have documents with many fields
  - Don't know all the fields in advance
  - Or want to change the default mapping for certain fields
- Use **dynamic templates** to define a field's mapping based on one of the following:
  - The field's data type
  - The name of the field
  - The path to the field

### Dynamic Templates Example

- Map any string field with a name that start with `ip*` as type IP:

```http
PUT my_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "ip_fields": {
          "match_mapping_type": "string",
          "match": "ip*",
          "mapping": {
            "type": "ip"
          }
        }
      }
    ]
  }
}
```