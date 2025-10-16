RUN
===

To run PostgreSQL (with sample data), Elastic (with sample data), and the "really ugly" OGCAPI-Records web interface:

```
https://github.com/GeoCat/gn5_ogcapi_records_demo.git
cd gn5_ogcapi_records_demo
docker-compose up
```

You will also also need to run GN5 (in your IDE or with maven).  USE JAVA 21.

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


To delete any changes to PostgreSQL and Elastic, erase their volumes:

```
 docker-compose down --volume
```
 

Helpful URLs
============

OGCAPI-RECORDS
--------------

NOTE: its really helpful to have a browser extension to view formatted JSON data!

landing page: 
http://localhost:7979/geonetwork/ogcapi-records

list collections:
http://localhost:7979/geonetwork/ogcapi-records/collections

info about the "main" collection: 
http://localhost:7979/geonetwork/ogcapi-records/collections/3bef299d-cf82-4033-871b-875f6936b2e2


query endpoint for  "main" collection:
http://localhost:7979/geonetwork/ogcapi-records/collections/3bef299d-cf82-4033-871b-875f6936b2e2/items

example record in "main" collection (ogcapi-record json):
http://localhost:7979/geonetwork/ogcapi-records/collections/3bef299d-cf82-4033-871b-875f6936b2e2/items/004571b9-4649-42b3-9c28-a8cdc2bf53c7


example record in "main" collection (dcat: eu-dcat-ap):
http://localhost:7979/geonetwork/ogcapi-records/collections/3bef299d-cf82-4033-871b-875f6936b2e2/items/004571b9-4649-42b3-9c28-a8cdc2bf53c7?f=application%2Frdf%2Bxml&profile=http%3A%2F%2Fgeonetwork.net%2Fdef%2Fprofile%2Feu-dcat-ap


List queryables:
http://localhost:7979/geonetwork/ogcapi-records/collections/3bef299d-cf82-4033-871b-875f6936b2e2/queryables


list facets:
http://localhost:7979/geonetwork/ogcapi-records/collections/3bef299d-cf82-4033-871b-875f6936b2e2/facets

list sortables:
http://localhost:7979/geonetwork/ogcapi-records/collections/3bef299d-cf82-4033-871b-875f6936b2e2/sortables


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

password is `postgres`


Troubleshooting
===============

Elasticsearch
-------------

If your disk is more than 80% full, Elasticsearch may not function properly. To work around this and loosen this check, use the command:

```
curl -X PUT "localhost:9200/_cluster/settings" -H 'Content-Type: application/json' -d' { "transient": { "cluster.routing.allocation.disk.watermark.low": "95%", "cluster.routing.allocation.disk.watermark.high": "97%", "cluster.routing.allocation.disk.watermark.flood_stage": "99%" } }'
```

If Elasticsearch fails to restore shards after restarting, try the following commands:

Close the index:

```
curl -X POST "localhost:9200/gn-records/_close?pretty"
```

Retry the restore:

```
curl -X POST "localhost:9200/_snapshot/my_backup/snapshot_1/_restore?pretty" -H 'Content-Type: application/json' -d'
{
  "indices": "gn-records"
}'
```
