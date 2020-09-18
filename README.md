# chained-watcher-demo

This is a demonstration of how an Elasticsearch watcher can be composed as a "chain" of different instructions. 

This particular demo provides a demonstration of a watcher that (1) reads documents from Elasticsearch, (2) executes a query based on the results from the first query, and (3) writes data back to Elasticsearch based on the data returned in (1) and (2). 

Note: this functionality would likely be better accomplished using [transforms](https://www.elastic.co/guide/en/elasticsearch/reference/current/transforms.html). Regardless, once this demo is understood then it should be relatively straightforward apply different chained logic to meet your own requirements.
