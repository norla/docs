## Terms

Elastic search + Logstash (log storage) + Kibana (dashboards/visualisation) = ELK stack.

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

- Index - database
- Type - table
- Document - row

## Getting

GET /index/type/id

response looks like

{
"\_index": "movies",
"\_type": "movie",
"\_id": "1",
"\_version": 2,
"found": true,
"\_source": { ... document here ... }
}

## Searching

GET/POST /index/type/\_search

Both index and type are optional

Body contains a query json object:

{
"query": ... query DSL here ...
"sort": ... sorting here ...
"size": ... page size ...
"from": .... start page ...
"\_source": ... list of json properties here ...
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
"regexp": {"somefield" : "[a-z]\*foobar"}
"terms"/"in": {"somefield" : ["val1", "val2", "val3"]}
"type": {"value": "sometype"}

"bool": {"must": [filter1, filter2, ...], must_not: [filter3, filter4, ...], "should" : [filter5, filter6, ...]}

- all "must" should match
- all "must_not" must not match
- at least one "should" should match

## Mapping

Each index has one "mapping type" which determines how new documents are indexed.
A mapping type has a list of fields for documents in the index. Each field has a
type. Multi-fields enable the same property be indexed in different ways.

Once a mapping has been defined for a field it cannot be changed. In that case the index need to be re-created.

## Dynamic templates

Instead of defining fields manually we can use dynamic mappings, which will
add new fields as the are detected in new documents. Beware of field explosion however.

For example we can say all properties of type string should be stored as "not analysed",elastic search term
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

To enable a dynamic mapping for all types in an index, set type to "\_default"

## Index templates

Index templates are automatically applied when new indices are created.
Templates contain both settings and mappings.

## Alias

The alias api allows aliasing an index with another name. All API:s will translate
the alias to the actual name in runtime.

Several indexes can have the same alias. They alias will then be expanded to all of them.

Can be used together with the reindex api to migrate schema wothout downtime:

1. Create alias "myalias" for "old_index"
2. Let application use "myalias", if not doing so already.
3. Create "new_index" with new mappings/settings
4. Copy data to new_index using the re-index api.
5. Add "new_index" to "myalias" and remove "old_index" in a single atomic op.

## Clustering

ES is distributed by nature.

Node - running instace of ES.
Cluster - one or more nodes with the same cluster.name

Node types:
Master - In charge of cluster-wide updates (add/remove indexes and nodes, allocating shards etc)
Data - Holds shards and perfoms peforms search, aggregation, CRUD
Ingest - Provides pre-processing of documents before indexing takes place.
Master Eligable - Hot-standby(s) in case master goes down.
Coordinating - Node that receives requests and delegates to involved data
node(s). All nodes are implicitly coordinators. A designated
coordinator is created by creating a node with master, data
and ingest set to "false".

Data is spread evenly across nodes.
Clients can talk to any node.
Whichever node a client talks to will talk to other nodes if needed to complete the request.

Index: A place to store data, logical namespace pointing to one ore more shards
Shard: Low-level worker unit that holds a slice of data for an index.
Each shard is an instance of Lucene and a complete search engine.
Documents are stored an indexed in shards, but clients never talk to them.
Each shard has one or more replica(s) for load balancing and fault tolerance.
The number of shards an index has limits the amount of data it can hold.

## Health

Green: All primary and replica shards are active
Yellow: All primary shards active, not all replica shards are active
Red: Not all primary shards are active.

## Backup / restore

Elastic search can create _snapshots_ of an index. Snapshots are incremental and therefore quite cheap to make if the are performed on a regular basis.

Every snapshot belongs to a _repository_. This can be a for example a shared fs, an azure disk or an s3 bucket.

### S3 Repositores

To take ES snapshots to S3, the _cloud-aws_ plugin should be installed.
This enables us to create repositories of type "s3".

```
curl -XPUT 'http://localhost:9200/_snapshot/s3_repository' -d'
{
  "type": "s3",
  "settings": {
    "bucket": "bucket-snapshot",
    "region": "eu-west-1",
    "access_key": "...",
    "secret_key": "..."
  }
}'
```

### Take a snapshot to a repository

```
curl -XPUT "http://localhost:9200/_snapshot/s3_repository/snap1?pretty?wait_for_completion=true" -d'
{
   "indices": "products, index_1, index_2",
    "ignore_unavailable": true,
    "include_global_state": false
}
```

(Defatult is to take a snapshot of ALL indices)

### Restoring from a snapshot:

```
curl -s -XPOST --url "http://localhost:9200/_snapshot/s3_repository/snap1/_restore" -d'
{
    "indices": "index_1,index_2",
    "ignore_unavailable": true,
    "include_global_state": false
}'
```
