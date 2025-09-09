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

password is `postgres`



