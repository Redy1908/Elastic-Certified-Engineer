### Documents are JSON Objects 

Given any kind of data, we need to rappresent it in a way that Elasticsearch can understand. Elasticsearch uses JSON (JavaScript Object Notation) to represent documents.

### Documents are Indexed into an Index

An index is:
- A logical way of grouping data
- Can be seen as an optimized collection of documents

### Index a Document: curl Example

To create an index, send a request using `POST` that specifies;
- index name
- _doc resource
- the document to be indexed

By default, Elasticsearch will generate a unique ID for the document

```bash
curl -X POST "localhost:9200/my-index/_doc/" -H 'Content-Type: application/json' -d'
{  
  "title": "My First Document",
  "content": "This is the content of my first document."
}
'
```

### Dev Tools

Using curl to interact with Elasticsearch can be tedious. Kibana's Dev Tools console provides a more user-friendly way to interact with Elasticsearch.

- Find it in: [Kibana > Management > Dev Tools > Console](https://127.0.0.1:5601/app/dev_tools#/console/shell)

- The syntax is similar to curl, but you don't need to specify the full URL or headers.

```http
POST /my-index/_doc/
{  
  "title": "My First Document",
  "content": "This is the content of my first document."
}
```

### Index a Document: PUT vs POST

When you index a document using:
- `PUT`: You specify the document ID. If a document with that ID already exists the index will be updated (overwritten) and the _version incremented by 1.
  - Example:
  ```http
  PUT /my-index/_doc/1
  ...
  ```
- `POST`: Elasticsearch generates a unique ID for the document.

### Retrieve a Document

Use the `GET API` to retrieve a document by specifying the index name and document ID.

```http
GET /my-index/_doc/1
```

### Create a Document

Index a new JSON document with the `_create` resource:
- Guarantees that a new document is only indexed if it does not already exist.
- Can not be used to update an existing document.

```http
PUT /my-index/_create/1
...
```

### Update Specific Fields

Use the `_update` resoource to modify specific fields in a document:
- Add the `doc` context
- `_version` is incremented by 1

```http
POST /my-index/_doc/1/_update
{
  "doc": {
    "content": "This is the updated content of my first document."
  }
}
```

### Delete a Document

Use the `DELETE` to delete an indexed document

```http
DELETE /my-index/_doc/1
```

### The Bulk API

- Use the `Bulk API` to index many documents in a single API call:
  - Increases indexing speed
  - Usefull if you need to index a data stream such as log events
- Four actions:
  - create, index, update and delete
- The response is a large JSON structure
  - Returns individual results of each action that was performed
  - Failure oof a single action does not stop the entire bulk operation

### Bulk API Example

Newline delimited JSON (NDJSON) structure:
- Increases the indexing speed
- Index, create, update actions expect a newline followed by a JSON object on a single line

```http
POST /my-index/_bulk
{ "index": {} }
{ "title": "Document 1", "content": "Content for document 1." }
{ "index": {"_id": 2} }
{ "title": "Document 2", "content": "Content for document 2." }
{ "update": { "_id": "3" } }
{ "doc": { "content": "Updated content for document 3." } }
{ "delete": { "_id": "2" } }
```

There is no correct number of actions to include in a single bulk request. Elasticsearch limit the maximum size of an HTTP request to 100MB. In cases where you are indexing large documents, you can preprocess your document in smaller batches to avoid exceeding this limit.

### Upload a File in Kibana

Quickly [uplaod](https://127.0.0.1:5601/app/home#/tutorial_directory/fileDataViz) a log file or delimited CSV, TSV, or JSON file:
- Used for initial exploration of data
- Not intended for production use cases

### Understanding data

- Most data con be categorized into:
  - (relatively) static data: data set that may grow or change, but slowly or infrequently (e.g., product catalog)
  - time-series data: event data associated with a moment in time that grows rapidly (e.g., log data, metrics data)
- Elastic Stack works well with both types of data