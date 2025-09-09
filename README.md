RUN
===

To run PostgreSQL (with sample data), Elastic (with sample data), and the "really ugly" OGCAPI-Records web interface:
```
docker-compose up
```

You will also also need to run GN5 (in your IDE or with maven).  USE JAVA 23.

```
git clone https://github.com/geonetwork/geonetwork.git

cd geonetwork
mvn clean install -Drelax

cd src/apps/geonetwork
mvn spring-boot:run
```

For use in Intellij:
1. checkout GN5 and build (see above)
2. In Intellij; <br>
    a. load the GN5 pom  <br>
    b. create a new spring-boot run/debug configuration <br>
    c. run the configuration

NOTES
-----

`docker-compose` will run:
1. postgresql (with sample data)<br>
    PORT: 5555<br>
    USER: postgres<br>
    PASS: postgres<br>
    DB: geonetwork
2. elastic (with sample data)<br>
    PORT: 9200<br>
    no user/pass<br>
    INDEX: gn-records
3. ogcapi-records angular web interface<br>
    PORT: 80<br>
    assumes GN5 running at http://localhost:7979


 

Helpful URLs
============

OGCAPI-RECORDS
--------------

NOTE: its really helpful to have a browser extension to view formatted JSON data!

landing page: 
http://localhost:7979/ogcapi-records

list collections:
http://localhost:7979/ogcapi-records/collections

info about the "main" collection: 
http://localhost:7979/ogcapi-records/collections/3bef299d-cf82-4033-871b-875f6936b2e2


query endpoint for  "main" collection:
http://localhost:7979/ogcapi-records/collections/3bef299d-cf82-4033-871b-875f6936b2e2/items

example record in "main" collection (ogcapi-record json):
http://localhost:7979/ogcapi-records/collections/3bef299d-cf82-4033-871b-875f6936b2e2/items/004571b9-4649-42b3-9c28-a8cdc2bf53c7


example record in "main" collection (dcat: eu-dcat-ap):
http://localhost:7979/ogcapi-records/collections/3bef299d-cf82-4033-871b-875f6936b2e2/items/004571b9-4649-42b3-9c28-a8cdc2bf53c7?f=application%2Frdf%2Bxml&profile=http%3A%2F%2Fgeonetwork.net%2Fdef%2Fprofile%2Feu-dcat-ap


List queryables:
http://localhost:7979/ogcapi-records/collections/3bef299d-cf82-4033-871b-875f6936b2e2/queryables


list facets:
http://localhost:7979/ogcapi-records/collections/3bef299d-cf82-4033-871b-875f6936b2e2/facets

list sortables:
http://localhost:7979/ogcapi-records/collections/3bef299d-cf82-4033-871b-875f6936b2e2/sortables


OpenAPI (swagger) doc:
http://localhost:7979/v3/api-docs


Web App
-------

http://localhost

ELASTIC
-------

Index Definition:

http://localhost:9200/gn-records

View actual elastic json index:

[http://localhost:9200/gn-records/_search?pretty=true&q=\*:\*&size=500](http://localhost:9200/gn-records/_search?pretty=true&q=*:*&size=500)

POSTGRESQL
----------

```
  psql -h localhost -U postgres -p 5555 -d geonetwork
```


<br><br><br><br><br><br><br><br><br><br>

HOW TO DUMP AND CREATE CONTAINERS
=================================

```
HIGHLY TECHNICAL AND IN NOTES FORM!!!

INFORMATION ON HOW TO CREATE NEW POSTGRESQL 
AND ELASTIC DUMPS.

YOU ONLY NEED TO READ THIS IF YOU WANT TO 
SAVE-AND-UPDATE THE SAMPLE DATA!
```



We need two docker containers;

1. postgresql (with loaded data)
2. elastic (with indexed data)
 



By-Hand Building (i.e. for upgrades)
------------------------------------

TODO: this should be automated

Get Running
%%%%%%%%%%%

1. run a docker postgresql (with the sample data injected)
   RUN FROM `docker/` (where `dump.gn.sql` is)
```
docker rm postgresql-gn5
docker run -d -e POSTGRES_DB=geonetwork -e POSTGRES_PASSWORD=postgres -e POSTGRES_USER=postgres -p 5555:5432  --name postgresql-gn5 -v `pwd`/dump.gn.sql:/docker-entrypoint-initdb.d/dump.gn.sql   postgres:16-alpine
```

2. run elastic
```
docker run --name gn5_demo-elasticsearch-1  -p 9200:9200 -p 9300:9300 -e "path.repo=/tmp" -e ES_JAVA_OPTS="-Xms750m -Xmx2g" -e "discovery.type=single-node" -e "xpack.security.enabled=false" -e "xpack.security.enrollment.enabled=false" docker.elastic.co/elasticsearch/elasticsearch:8.14.0
```


3. gn4 (from source)

Likely need to sign on as admin, then admin tools -> delete and reindex.

```
cd src/web
mvn jetty:run jetty:run -Dgeonetwork.db.type=postgres-postgis -Djdbc.database=geonetwork -Djdbc.username=postgres -Djdbc.password=postgres -Djdbc.host=localhost -Ddb.port=5555 -Djdbc.port=5555 -Djetty.port=8080
```  

4. validate working in gui

Dump Data
--------

Postgresql:

```
pg_dump -h localhost -p 5555 -U postgres --column-inserts geonetwork > dump.gn.sql
```

`pg_dump` version miss-match?  Use:
``` 
brew install postgresql@16
brew link --force --overwrite postgresql@16
```

Elastic - create dump
=====================

startup:
```
docker run --name gn5_demo-elasticsearch-1  -p 9200:9200 -p 9300:9300 -e "path.repo=/tmp" -e ES_JAVA_OPTS="-Xms750m -Xmx2g" -e "discovery.type=single-node" -e "xpack.security.enabled=false" -e "xpack.security.enrollment.enabled=false" docker.elastic.co/elasticsearch/elasticsearch:8.14.0
```

Run GN4 and admin->tools->delete index/re-index
* this will populate the Elastic index
* Verify by going to:
  http://localhost:9200/gn-records
  http://localhost:9200/gn-records/_search?pretty=true&q=*:*  (show records)

Create backup repo:
```
curl -XPUT -d '
{
"type": "fs",
"settings": {
"location": "/tmp/es_backups",
"compress": true
}
}' -H 'Content-Type: application/json' "http://localhost:9200/_snapshot/my_backup"
```


delete any existing repo:
```
curl -XDELETE "http://localhost:9200/_snapshot/my_backup"
```


delete any existing snapshot:
```
curl -XDELETE "http://localhost:9200/_snapshot/my_backup/snapshot_1"
```

Create actual backup:
```
curl -XPUT -d '
{
"indices": "gn-records",
"ignore_unavailable": true,
"include_global_state": false
}' -H 'Content-Type: application/json' "http://localhost:9200/_snapshot/my_backup/snapshot_1?wait_for_completion=true"
```

TAR up the backup:

```
docker exec -it gn5_demo-elasticsearch-1 bash

cd /tmp
tar cvfz es_backups.tar.gz es_backups/
```

download the backup

```
docker cp gn5_demo-elasticsearch-1:/tmp/es_backups.tar.gz .
```

Restore
=======

NOTE: order is imporant - repo must be created AFTER snapshot is put (decompressed) into the container

restart elastic (it will be empty)
```
docker kill gn5_demo-elasticsearch-1
docker rm gn5_demo-elasticsearch-1
docker run --name gn5_demo-elasticsearch-1  -p 9200:9200 -p 9300:9300 -e "path.repo=/tmp" -e ES_JAVA_OPTS="-Xms750m -Xmx2g" -e "discovery.type=single-node" -e "xpack.security.enabled=false" -e "xpack.security.enrollment.enabled=false" docker.elastic.co/elasticsearch/elasticsearch:8.14.0
```

copy old snapshot/backup to elastic container and un-tar it:
```
docker cp  es_backups.tar.gz gn5_demo-elasticsearch-1:/tmp/es_backups.tar.gz
docker exec   -w /tmp gn5_demo-elasticsearch-1 tar -xvzf es_backups.tar.gz
```

Create backup repo:
```
curl -XPUT -d '
{
"type": "fs",
"settings": {
"location": "/tmp/es_backups",
"compress": true
}
}' -H 'Content-Type: application/json' "http://localhost:9200/_snapshot/my_backup"
```



load snapshot into elastic
```
curl -XPOST -d '
{
  "indices": "*",
  "ignore_unavailable": true,
  "include_global_state": false
}' -H 'Content-Type: application/json' "http://localhost:9200/_snapshot/my_backup/snapshot_1/_restore"
```


Helpful Info
============

http://localhost:7979/ogcapi-records<br>
http://localhost<br>
http://localhost:9200/gn-records <br>
[http://localhost:9200/gn-records/_search?pretty=true&q=\*:\*&size=500](http://localhost:9200/gn-records/_search?pretty=true&q=*:*&size=500)<br>
`docker exec -it gn5_demo-elasticsearch-1 bash`<br>
