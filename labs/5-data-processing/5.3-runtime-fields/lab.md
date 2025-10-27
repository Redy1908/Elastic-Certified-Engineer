
### Scripting

- Whenever scriptiin is supported in the Elsaticsearch APis, the syntax follows the same pattern
- Elasatichsearch compiles new scripts and stores the compiled version in a cache

```console
{
  "script": {
    "lang": "...",
    "source": | "id": "...",
    "params": { ... }
  }
}
```

- Scripts may be inline or stored
- Use the "source" for inline scripts or "id" for stored scripts
- Pass "params" instead of hardcoding values

### Painless Scripting

- Painless is a performant, secure scripting langueage designed specifically for Elasticsearch
- Paineless is the default language
  - You don't need to specify the language if you are wroting painless scripts
- Use Painless to
  - Process reindexed data
  - Create runtime fields which are evaluated at query time

### Example of a Painless Script

- Painless has a Java-like syntax (and can contain actual Java code)
- Fields of a document can be accessed using a **Map** named **doc** or **ctx**

```http
PUT _ingest/pipeline/url_parrser
{
  "processors": [
    {
      "script": {
        "source": "ctx['new_field'] = ctx['url'].splitOnToken('/')[2]"
      }
    }
  ]
}
```

### "Schema on read" with Runtime Fields

- Ideally, your schema is defined at index time ("schema on write")
- However, there are situations, where you may want to define a schema on read:
  - To fix errors in your data
  - To sstructure or parse your data
  - To change the way Elasticsearch returns data
  - To add new fields to your docuements

... without having too reindex your data

### Creating Runtime Fields in Kibana

- Configure
  - a name for the field
  - the field type
  - a custom label (optional)
  - a description (optional)
  - a value (optional)
  - a format (optional)

### Runtime Fields and Painless Tips

- Avoid runtime fields if you can
  - They are computationally expensive
  - Fix data at ingest time instead
- Avoid errors by checking for null values
- Use the Preview pane to validate your script

### Mapping a Runtime Field

- **runtime** section defineis the field **in the mapping**
- Use a Painless script to **emit** a field of a given type

```http
POST blogs/_mapping
{
  "runtime": {
    "day_of_week": {
      "type": "keyword",
      "script": {
        "source": emit(doc['publish_date'].value.getDayOfWeekEnum().toString())
      }
    }
  }
}
```

### Searching a Runtime Field

- You access runtime fields from the search API like any other field
  - Elasticsearch sees runtime fields no differently than regular fields

```http
GET blogs/_search
{
  "query": {
    "match": {
      "day_of_week": "MONDAY"
    }
  }
}
```

### Runtime Fields in a Search Request

- **runtime_mappings** section defines the field at **query time**
- Use a Painless script to **emit** a field of a given type

```http
GET blogs/_search
{
  "runtime_mappings": {
    "day_of_week": {
      "type": "keyword",
      "script": {
        "source": emit(doc['publish_date'].value.getDayOfWeekEnum().toString())
      }
    }
  },
  "aggs": {
    "my_agg": {
      "terms": {
        "field": "day_of_week"
      }
    }
  }
}
```