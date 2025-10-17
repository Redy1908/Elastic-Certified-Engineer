### Have You Noticed...

What do you think these queries return?
- different results?
- same results?

```http
GET blogs/_search
{
  "query": {
    "match": {
      "title": "Elasticsearch Basics"
    }
  }
}
```

```http
GET blogs/_search
{
  "query": {
    "match": {
      "title": "elasticsearch basics"
    }
  }
}
```

...that your text searches seems to be case-insensitive? Or that punctuation doesn't seem to matter?
- This is due to a process called **text analysis** which occurs when your string fields are indexed.

The above queries will return the same hits.

### Analysis Makes Text Searchable

- By default, text analysis breaks up a text string into individual words (called **tokens**) and lowercases them.
- Both at query time and at index time, the same analysis process is applied.

### Analyzers

- Text analysis is performed by components called **analyzers**.
- By default, Elasticsearch uses the **standard analyzer** which.
- There are many other built-in analyzers, including:
  - whitespace
  - stop
  - pattern
  - simple
  - language-specific analyzers (e.g., english, french, german, etc.)
  - and more...
- The built-in analyzers work well for many use cases
  - You can also create your own custom analyzers

### Anatomy of an Analyzer

An analyzer is made up of three main components:
- Zero or more **character filters**: pre-process the text before tokenization
- Exactly one **tokenizer**: breaks the text into tokens
- Zero or more **token filters**: process the tokens

### The Standard Analyzer

- The default analyzer
- No character filters
- Uses the standard tokenizer
- Lowercase all tokens
- Optionally removes stop words (common words like "the", "is", "and", etc.)

### Testing an Analyzer

Use the `_analyze` endpoint to see how an analyzer processes text.

```http
GET _analyze
{
  "analyzer": "english",
  "text": "The Quick Brown Foxes!"
}
```

This will return the tokens:
- quick
- brown
- fox

### Some Strings do not Need Analysis

- Text analysis is great for full text search
  - Enables you to search for individual words in a case-insensitive manner
- However, it is not great for things like aggregations
  - When you often want to see the original string

Example: You most likely want to search for `United States` instead of just `united` or `utates` when filtering by country.

### Keyword vs. Text

Elasticsearch provides two main string data types:
- `text`: analyzed strings for full text search
- `keyword`: not analyzed strings for exact matches, filtering, and aggregations
  - The original strings as they occur in the documents