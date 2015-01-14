# Docker Image for Titan Graph Database

1/16/2015 updated run.sh to update template for rexster
This image is based on the fine work done by apobbati on the apobbati/titan-rexster
Only difference is it is using 0.5.2: of Titan instead of 0.5.0 and will soon include gremlin-scala and some 
sample data

Titan is a free, open source database that is capable of processing
extremely large graphs and it supports a variety of indexing and storage backends,
which makes it easier to extend than some popular NoSQL Graph databases.

This docker image instantiaties a Titan graph database that is capable of
integrating with an ElasticSearch container (Indexing) and a Cassandra container (Storage).

The default distribution of Titan runs on a single node, so I thought it would be helpful
if there was a modular way at runtime to hook up Titan to its dependencies.

Enter Docker. Now it is possible to run Titan and it's dependencies in separate Docker containers.

## Titan

This container is using Titan 0.5.2. Please refer to
its [page](https://github.com/thinkaurelius/titan/wiki/Downloads) for more information.

## Tinkerpop and Rexster

[Tinkerpop](http://www.tinkerpop.com/) is a vendor-independent API specification for
manipulating and access Graph databases.

[Rexster](https://github.com/tinkerpop/rexster/wiki) is a service that provides protocols
for accessing a graph database. Currently it supports two protocols:
	- REST over HTTP: Human-readable and good for testing
	- RexPro: Binary Protocol for performence

If you'd like to avoid vendor lock-in, then I'd recommend using Rexster as the API
for accessing your Graph database. It has support for popular graph databases,
so you can avoid refactoring your code. Take a look at Tinkerpop Gremlin for a
Groovy-DSL for querying graphs to see how RexPro and Gremlin provide syntactical
elegance to query graphs.

## Running

if you have fig installed you can:

create fig.yml or get it from github repository for this image

cassandra:
  image: poklet/cassandra:latest
  expose:
    - "9160"
elasticsearch:
  image: dockerfile/elasticsearch
  expose:
    - "9200"
titan:
  image: ecompositor/titan-rexster-fig
  ports:
    - "8182"
    - "8183"
    - "8184"
  links:
    - cassandra
    - elasticsearch

 
fig up -d 

this will use the fig.yml file to serve as shorthand for the following run commands.


The minimum system requirements for this stack is 1 GB with 2 cores.

```
docker run -d --name es1 dockerfile/elasticsearch
docker run -d --name cs1 poklet/cassandra
docker run -d -P --name mytitan --link es1:elasticsearch --link cs1:cassandra ecompositor/titan-rexster-rig
```

### Ports

8182: HTTP port for REST API

8183: RexPro for native access (Binary protocol)

8184: JMX Port (You won't need to use this, probably)

You can read more about it in the Rexster documentation.

To test out the REST API (over Boot2docker):

```
curl http://localhost:<port-mapped-to-8182>/graphs/graph/vertices
```

## Dependencies

I've tested this container with the following containers:

	- poklet/cassandra: This is the Cassandra Storage backend for Titan. It scales well for large datasets.
	- dockerfile/elasticsearch: This is the ElasticSearch Indexing backend for Titan. It provides search
		capabilities for Titan graph datasets.

## Roadmap

In the near future, I'd like to add support for:

	- Scaling/Clustering Cassandra and ElasticSearch backends.
	- External volumes for persistent data.
	- Security between Titan and its backends.
	- Example application stack integrating with Titan.

