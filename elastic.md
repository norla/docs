## Terms

Elastic search + Logstash (log storage) + Kibana (dashboards/visualisation) = ELK stack.

Marvel = commercial plugin for elastic search.
Sense = Elastic search client, part of Marvel.

## Indexing

Create or Updtate a document ( = json object)

PUT /index/type/id will update or create document with given id.
POST /index/type/ will create a new document and generate an id for it.

Reply looks like:
```
{
  "_index": "movies",
  "_type": "movie",
  "_id": "1",
  "_version": 2,
  "created": false
}
```

Supply a "version" parameter when indexing and ES will do optimistic locking for you.

## Data structure

Each type has it's own id space
Different types have different mappings
Easy to search for specific type(s)

SQL analogy:
* Index - database
* Type - table
* Document - row

## Getting

GET /index/type/id
    
response looks like


{
  "_index": "movies",
  "_type": "movie",
  "_id": "1",
  "_version": 2,
  "found": true,
  "_source": { ... document here ... }
}

## Searching

GET/POST /index/type/_search

Both index and type are optional

Body contains a query json object:

{
  "query": ... query DSL here ...
  "sort": ... sorting here ...
  "size": ... page size ...
  "from": .... start page ...
  "_source": ... list of json properties here ...
  "aggregations: ... aggregations here ...
}

## Query DSL

NOTE: All examples below should be embedded in {"query": ... stuff goes here ...}

"query_string": free text search on keywords, several words will be OR:ed 

```
"query_string": {
   "query": "Francis Ford Coppola",
   "fields": ["title"]       // Optional
   "default_operator": "AND" //Optional
}
```

"filtered": limit results of query further with filtering

```
 "filtered": {
   "query": ... any query here ...
   "filter": {
      "terms": {"year": 1962}
    }
}
```

The "term" filter searches for documents with matching json properties.

Queying scores the results relevance, while filtering does not.
Filters are therefore faster and can be cached. Always use filtering
unless you need results sorted by relevance.

A nice shorthand to use when you only wish to filter and not query:

```
"constant_score": {
   "filter": {
      "term": { "year": 1962 }
   }
}
```
Several filters in addition to "term" are available:

"and": [filter1, filter2...]
"or": [filter1, filter2]
"exists": {"field": "somefieldname"}
"missing": {"field": "somefieldname"}
"not": filter
"prefix": { "somefield" : "somestring" }
"range": {"somefield": {"gt/gte": "somevalue", ""lt/lte": "someothervalue"}}
"regexp": {"somefield" : "[a-z]*foobar"}
"terms"/"in": {"somefield" : ["val1", "val2", "val3"]}
"type": {"value": "sometype"}

"bool": {"must": [filter1, filter2, ...], must_not: [filter3, filter4, ...], "should" : [filter5, filter6, ...]}
* all "must" should match
* all "must_not" must not match
* at least one "should" should match


## Mapping

When a document is indexed two things happen:

1. The original json data is stored as-is (as seen in "_source")
2. Each json property is indexed to one or more fields in a lucene index.

The lucene indexing of json properties is by default based on the type of field.
Strings are by default tokenized and lower cased which often causes confusion.

PUT /index/type/_mapping

Sometimes it is convinient to add more than one mapping to a json property.
This is called a "multimapping". Each mapping has it's own name and you specify
which one you want to use using the "field" in queries/filters.

## Dynamic mapping

Changing the field for each property to suit your needs can be very cumbersome.
Dynamic mappings are templates that ES looks for when indexing a field it has not
seen before.

For example we can say all properties of type string should be stored as "not analysed",
meaning that they are not tokenized or decapitalized when indexed.

```
PUT /movies/_mapping/movie
{
  "movie": {
    "dynamic_templates": [
      {
        "strings": {
          "match_mapping_type": "string",
          "path_match": "*",
          "mapping": {
            "type": "string",
            "fields": {
              "original": {
                "type": "string",
                "index": "not_analyzed"
```

To enable a dynamic mapping for all types in an index, set type to "_default"


## Clustering

ES is distributed by nature.

Node - running instace of ES.
Cluster - one or more nodes with the same cluster.name
Master - In charge of cluster-wide updates (add/remove indexes and nodes, etc)

Data is spread evenly across nodes.
Clients can talk to any node. 
Whichever node a client talks to will talk to other nodes if needed to complete the request.

## Health

Green: All primary and replica shards are active
Yellow: All primary shards active, not all replica shards are active
Red: Not all primary shards are active.

Index: A place to store data, logical namespace pointing to one ore more shards
Shard: Low-level worker unit that holds a slice of data for an index.
       Each shard is an instance of Lucene and a complete search engine.
       Documents are stored an indexed in shards, but clients never talk to them.
       Each shard has one or more replica(s) for load balancing and fault tolerance.
       The number of shards an index has limits the amount of data it can hold.
