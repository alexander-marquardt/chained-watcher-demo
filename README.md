# chained-watcher-demo

This is a demonstration of how an Elasticsearch watcher can be composed as a "chain" of different instructions. 

This particular demo provides a demonstration of a watcher that (1) reads documents from Elasticsearch, (2) executes a query based on the results from the first query, and (3) writes data back to Elasticsearch based on the data returned in (1) and (2). 

Note: this functionality would likely be better accomplished using [transforms](https://www.elastic.co/guide/en/elasticsearch/reference/current/transforms.html). Regardless, once this demo is understood then it should be relatively straightforward apply different chained logic to meet your own requirements.

# Ensure the range query has a sensible value
Be careful to set the range of the initial query to a sensible value to avoid pulling back too much data. Something like 10 minutes would be more sensible than 60 days for most use cases. 

```
"range": {
    "@timestamp": {
        "gte": "now-60d"
    }
}
```

# Loading data
For this demo, you can load a few documents with commands similar to the following. Be sure to put timestamps that make sense for the current range query used by the watcher.

```
POST transaction_original/_doc
{"message": "abc", "ident": "id1", "@timestamp": "2020-08-18T19:43:36.000Z", "other_field": "other_val 1", "tags": ["start_event"]}

POST transaction_original/_doc
{"message": "def", "ident": "id1", "@timestamp": "2020-08-18T19:53:36.000Z", "other_field": "other_val 2", "tags": ["end_event"]}

POST transaction_original/_doc
{"message": "ghi", "ident": "id2", "@timestamp": "2020-08-20T19:43:56.000Z", "other_field": "other_val 4", "tags": ["end_event"]}

POST transaction_original/_doc
{"message": "jkl", "ident": "id2", "@timestamp": "2020-08-20T19:43:36.000Z", "other_field": "other_val 3", "tags": ["start_event"]}
```


