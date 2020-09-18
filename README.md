# chained-watcher-demo

This is a demonstration of how an Elasticsearch watcher can be composed as a "chain" of different instructions. 

This particular demo provides a demonstration of a watcher that reads documents from Elasticsearch, executes a query based on the results from the first query, and writes data back to Elasticsearch. 

Specifically, the following steps are executed
1. Queries for documents that have a tag of "end_event" within the past X minutes. 
2. Composes a list of "ident" values frome each of the returned documents.
3. Executes a terms query for all documents that have a tag of "start_event" that have the same "ident" values returned in the previous step. 
4. Create a hash map of of all of the "start_event" documents that are returned, with "ident" as the key. 
5. Loop over all of the "end_event" docs (from step 1), and lookup the "ident" in the hash map from step 4. Calculate the difference between the "start_event" timestamp, and the "end_event" timestamp. Store any modified documents in an array that will contain documents to be overwritten.
6. Execute the "index" action to send the array of documents to Elasticsearch. 

Note: this functionality would likely be better accomplished using [transforms](https://www.elastic.co/guide/en/elasticsearch/reference/current/transforms.html) or the approach documented in [Using Logstash and Elasticsearch scripted upserts to calculate transaction duration from out-of-order events](https://alexmarquardt.com/2020/09/16/using-logstash-and-elasticsearch-scripted-upserts-to-calculate-transaction-duration-from-out-of-order-events/). Regardless, once this demo is understood then it should be relatively straightforward apply different chained logic to meet your own requirements.

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


