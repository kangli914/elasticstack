# Elastic Stack Notes

Notes taken from learning Elasticsearch Stack

## Miscellaneous APIs

### /_cat APIs:

##### CAT API to view cluster:
Cluster is categorized by name and default is *elastisearch*.

Append *v* to the end of the call to include header.

Append *pretty* to the end of the call to tell it to pretty-print the JSON response if any.
```
GET /_cat/health?v
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1556390894 18:48:14  elasticsearch yellow          1         1      6   6    0    0        5             0                  -                 54.5%

GET /_cat/health?pretty
1556390837 18:47:17 elasticsearch yellow 1 1 6 6 0 0 5 0 - 54.5%

GET /_cat/health?v&pretty
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1556390859 18:47:39  elasticsearch yellow          1         1      6   6    0    0        5             0                  -                 54.5%


curl -XGET "http://localhost:9200/_cat/health?v&pretty"
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1556390917 18:48:37  elasticsearch yellow          1         1      6   6    0    0        5             0                  -                 54.5%
```

##### CAT API to view node:
Note node name is randomly assgined as *nWcm8Fn* 
```
GET /_cat/nodes?v
curl -X GET "localhost:9200/_cat/nodes?v"

ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
127.0.0.1           20          48  17                          mdi       *      nWcm8Fn

```

##### CAT API to view indices:
Each index has assgined *uuid*. *Primmary* shards and *replicas* shards are default **[5]** and **[1]** in v6.5.4.
Primary shards will be **[1]** in v7.0.0
```
GET _cat/indices?v
curl -XGET "http://localhost:9200/_cat/indices?v&pretty"

health status index     			   uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .kibana_1                gIM8MubdSQuRFiCJnzb5Hw   1   0          8            0     45.4kb         45.4kb

yellow open   dummybk                  5CujTF1STYKgIxiAdumvOQ   5   1       1000            0    475.1kb        475.1kb
yellow open   dummylogstash-2019.03.13 c9ZEvldFSv-hEZTmGqEJng   5   1       4624            0     21.3mb         21.3mb
yellow open   dummylogstash-2019.03.14 jljI5sxAQAaZbhQuMloaPw   5   1       4750            0     21.9mb         21.9mb
yellow open   dummylogstash-2019.03.12 Ix9ZKIf4SxOFx5QZv64iJw   5   1       4631            0     21.1mb         21.1mb
``` 

### /_search APIs: 
##### Search API (or GET API in Document API) by size
```
http://localhost:9200/bank/account/_search?size=50
GET bank/account/_search?size=50
```
1 - Simple query string as a parameter - sending search parameters **through the REST request URI** 
	
```	 
GET bank/account/_search?q=firstname:Virginia
$ curl -XGET "http://localhost:9200/bank/account/_search?q=firstname:Virginia"
```
response:
```
{"took":1,"timed_out":false,"_shards":{"total":5,"successful":5,"skipped":0,"failed":0},"hits":{"total":1,"max_score":4.882802,"hits":[{"_index":"bank","_type":"account","_id":"25","_score":4.882802,"_source":{"account_number":25,"balance":40540,"firstname":"Virginia","lastname":"Ayala","age":39,"gender":"F","address":"171 Putnam Avenue","employer":"Filodyne","email":"virginiaayala@filodyne.com","city":"Nicholson","state":"PA"}}]}}
```	 
Sorting example after search:
```
GET /bank/_doc/_search?sort=account_number:asc
curl -XGET "http://localhost:9200/bank/_doc/_search?sort=account_number:asc"
```
	
2 - Using a request body - sending search parameters **through the REST request body**:  The search request can be executed with a search DSL, 
which includes the Query DSL, within its body
```
GET bank/account/_search/
{
  "query": { "match": {"firstname": "Virginia"}}
}	

$ curl -XGET "http://localhost:9200/bank/account/_search/" -H 'Content-Type: application/json' -d'
{
   "query": { "match": {"firstname": "Virginia"}}
}'
```
Sorting example:
```
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [	{ "account_number": "asc" } ],
  "size": 100
}

$ curl -XGET "http://localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [ { "account_number": "asc"} ],
  "size": 100
}'
```


### Action *PUT* && /_update APIs:
##### PUT to create indices:
1 - Default number of shards will change from **[5]** in v6.5.4 to **[1]** in v7.0.0:

2 - Elasticsearch by default created **[1]** replica for this index

3 - Yellow means all data is available but some replicas are not yet allocated.The reason this happens for this index is because Elasticsearch by default created one replica for this index. Since we only have one node running at the moment, that one replica cannot yet be allocated (for high availability) until a later point in time when another node joins the cluster. Once that replica gets allocated onto a second node, the health status for this index will turn to green.

```
PUT /customer

6.5.4
-----
GET /_cat/indices/customer?v
health   status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
*yellow* open   customer HWoWx81aRDiUfhei3kTWvw   5   1          0            0      1.1kb          1.1kb
7.0.0
-----
health   status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
*yellow* open   customer 95SQ4TSUT7mWBT7VNHH67A   1   1          0            0    
```

##### PUT to [Create|Replace] documents:
1 - if Document doesn't exist then PUT can be used as creating document

2 - PUT update whole documents instead of partial update. For example, 2nd PUT will not only update the name but also erase the content in documents. e.g. as *Replacing* original document

```
PUT /customer/_doc/1?pretty
{
  "name": "John Doe",
  "contact": 123456
}
curl -XPUT "http://localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "John Doe",
    "contact": 123456
}'

GET /customer/_doc/_search?
```
*response:*
```
"hits" : [
  {
	"_index" : "customer",
	"_type" : "_doc",
	"_id" : "1",
	"_score" : 1.0,
	"_source" : {
	  "name" : "John Doe",
	  "contact" : 123456
	}
  }
]
```
```
PUT /customer/_doc/1?pretty
{
  "name": "Air flow"
}

GET /customer/_doc/1
```

*response:*
```
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 14,
  "found" : true,
  "_source" : {
    "name" : "Air flow"
  }
}
```

##### Update API to update document:
unlike **PUT** action, **/_update** only changing document with the name field
below _update preserves 'age' field

There are 2 ways to update the document:

1 - 'doc'
```
POST /customer/_doc/1?pretty
{
  "name": "Air flow",
  "age": 29
}
GET /customer/_doc/1
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 75,
  "found" : true,
  "_source" : {
    "name" : "Air flow",
    "age": 29
  }
}

POST /customer/_doc/1/_update?pretty
{
  "doc": { "name": "Jane Doe" }
}
GET /customer/_doc/1
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 76,
  "found" : true,
  "_source" : {
    "name" : "Jane Doe",
    "age" : 29
  }
}
```
2 - 'script'
```
POST /customer/_doc/1/_update?pretty
{
  "script" : "ctx._source.age += 5"
}

POST /customer/_doc/1/_update?pretty
{
  "script" : "ctx._source.name = 'Avatar'"
}
```



### Prerequisites

What things you need to install the software and how to install them

```
Give examples
```

### Installing

A step by step series of examples that tell you how to get a development env running

Say what the step will be

```
Give the example
```

And repeat

```
until finished
```

End with an example of getting some data out of the system or using it for a little demo

## Running the tests

Explain how to run the automated tests for this system

### Break down into end to end tests

Explain what these tests test and why

```
Give an example
```

### And coding style tests

Explain what these tests test and why

```
Give an example
```

## Deployment

Add additional notes about how to deploy this on a live system

## Built With

* [Dropwizard](http://www.dropwizard.io/1.0.2/docs/) - The web framework used
* [Maven](https://maven.apache.org/) - Dependency Management
* [ROME](https://rometools.github.io/rome/) - Used to generate RSS Feeds

## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/your/project/tags). 

## Authors

* **Billie Thompson** - *Initial work* - [PurpleBooth](https://github.com/PurpleBooth)

See also the list of [contributors](https://github.com/your/project/contributors) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

