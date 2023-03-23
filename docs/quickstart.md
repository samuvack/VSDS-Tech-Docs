---
sort: 1
---


# QUICK START

## Example 1

```note
**_Please Note!_**

_The quick start try-out is ONLY made for making _[Pagination fragmentation](https://github.com/Informatievlaanderen/VSDS-LDESServer4J/tree/main/ldes-fragmentisers/ldes-fragmentisers-pagination)_ for the self-generated data type: _<https://www.w3.org/TR/vocab-ssn-ext/#sosa:ObservationCollection>_. To support other fragments, data types, or other features e.g., retention, caching, etc., please consult _[LDES Server Manual](https://github.com/Informatievlaanderen/VSDS-LDESServer4J)_ for the configurations._

[Vlaamse Smart Data Space](https://www.vlaanderen.be/digitaal-vlaanderen/onze-oplossingen/vlaamse-smart-data-space)_ projects also provide methods for transforming data to _[LDES format](https://semiceu.github.io/LinkedDataEventStreams/)_, e.g. from _[NGSI-V2](https://vloca-kennishub.vlaanderen.be/NGSI-v2)_ to _[NGSI-LD](https://en.wikipedia.org/wiki/NGSI-LD)_, from NGSI to _[OSLO](https://www.vlaanderen.be/digitaal-vlaanderen/onze-oplossingen/oslo)_ Model, from _[NGSI-V2](https://vloca-kennishub.vlaanderen.be/NGSI-v2)_ to _[LDES (LinkedDataEventStreams)](https://semiceu.github.io/LinkedDataEventStreams/)_ etc. For more info, please consult: _[VSDS Linked Data Interactions](https://github.com/Informatievlaanderen/VSDS-Linked-Data-Interactions)
```

In this short example below, we show you how an LDES server publishes your first [LDES collection](https://semiceu.github.io/LinkedDataEventStreams/) and LDES client can link domains and systems using LDES. The whole story can be read here: <https://medium.com/p/e1aab3090b40/edit>

### Before starting

1.  [Docker](https://docker.com/) has been installed on your device.
2.  Local Ports 8080, and 27017 are accessible.
3.  The command script is written in [bash](https://en.wikipedia.org/wiki/Bash_%28Unix_shell%29) code. Please modify it accordingly.

### Start your LDES server

1.  Create a local [docker-compose.yml](https://stackedit.io/docker-compose.yml) file with the following content or you could use the current good-to-go [GetStarted_VSDS Github project](https://github.com/xdxxxdx/GetStarted_VSDS):

```
version: ‘3.3’
services:
  ldes-server:
    container_name: quick-start_ldes-server
    image: ghcr.io/informatievlaanderen/ldes-server:latest
    environment:
      - SIS_DATA=/tmp
      - SPRING_DATA_MONGODB_DATABASE=sample
      - LDES_COLLECTIONNAME=sample
      - LDES_MEMBERTYPE=https://www.w3.org/TR/vocab-ssn-ext/#sosa:ObservationCollection
      - SPRING_DATA_MONGODB_HOST=ldes-mongodb
      - SPRING_DATA_MONGODB_PORT=27017
      - LDES_HOSTNAME=http://localhost:8080
      - LDES_SHAPE=
      - VIEW_TIMESTAMPPATH=http://www.w3.org/ns/prov#generatedAtTime
      - VIEW_VERSIONOFPATH=http://purl.org/dc/terms/isVersionOf
      - VIEWS_0_FRAGMENTATIONS_0_NAME=pagination
      - VIEWS_0_NAME=by-page
      - VIEWS_0_FRAGMENTATIONS_0_CONFIG_MEMBERLIMIT=1
    ports:
      - 8080:8080
    networks:
      - ldes
    depends_on:
      - ldes-mongodb
  ldes-mongodb:
    container_name: quick-start_ldes-mongodb
    image: mongo:6.0.4
    ports:
      - 27017:27017
    networks:
      - ldes
  ldes-cli:
    image: ghcr.io/informatievlaanderen/ldes-cli:20230222T0959
    container_name: quick-start_ldes-client-cli
    command: "--url http://localhost:8080/sample/by-page --input-format text/turtle"
    profiles:
      - delayed-started
    network_mode: service:ldes-server
networks:
  ldes:
    name: quick_start_network

```

2. Within the working directory of [docker-compose.yml](https://xdxxxdx.github.io/GetStarted_VSDS/docker-compose.yml), please\
   run docker compose up to start the [LDES Server](https://github.com/Informatievlaanderen/VSDS-LDESServer4J) and [MongoDB](https://www.mongodb.com/) containers.

3. **Now!** [LDES Server](https://github.com/Informatievlaanderen/VSDS-LDESServer4J) is running at port 8080 and ready for making [Pagination fragmentation](https://github.com/Informatievlaanderen/VSDS-LDESServer4J/tree/main/ldes-fragmentisers/ldes-fragmentisers-pagination) for your data. Please. use your preferred browser to reach <http://localhost:8080/sample> to have a look.

_The result should be as follow:_

```
@prefix ldes:   <https://w3id.org/ldes#> .
@prefix rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix sample: <http://localhost:8080/sample/> .
@prefix tree:   <https://w3id.org/tree#> .
<http://localhost:8080/sample>
        rdf:type   ldes:EventStream ;
        tree:view  sample:by-page .
sample:by-page  rdf:type  tree:Node .
```

### Publish your first [LDES collection](https://semiceu.github.io/LinkedDataEventStreams/)

1.  Create your own `sample.ttl` file with the following content, or you could use the current good-to-go [GetStarted_VSDS Github project](https://github.com/xdxxxdx/GetStarted_VSDS):

```
version: '3.3'<br>
services:<br>
  ldes-server:<br>
 container_name: quick-start_ldes-server<br>\ image: ghcr.io/informatievlaanderen/ldes-server:latest<br>\
    environment:<br>

 - SIS_DATA=/tmp<br>\
      - <br>SPRING_DATA_MONGODB_DATABASE=sample<br>\ - LDES_COLLECTIONNAME=sample<br>\
      - <br>LDES_MEMBERTYPE=https://www.w3.org/TR/vocab-ssn-ext/<br>#sosa:ObservationCollection<br>\ - <br>SPRING_DATA_MONGODB_HOST=ldes-mongodb<br>\
      - SPRING_DATA_MONGODB_PORT=27017<br>\
      - <br>LDES_HOSTNAME=http://localhost:8080<br>\
      - LDES_SHAPE=<br>\
      - <br>VIEW_TIMESTAMPPATH=http://www.w3.org/ns/prov#generatedAtTime<br>\
      - <br>VIEW_VERSIONOFPATH=http://purl.org/dc/terms/isVersionOf<br>\
      - VIEWS_0_FRAGMENTATIONS_0<br>_NAME=pagination<br>\
      - VIEWS_0_NAME=by-page<br>\
      - VIEWS_0_FRAGMENTATIONS_0<br>_CONFIG_MEMBERLIMIT=1<br>\
    ports:<br>\
      - 8080:8080<br>

    networks:<br>\
      - ldes<br>\
    depends_on:<br>\
      - ldes-mongodb<br>\
  ldes-mongodb:<br>\
    container_name: quick-start_ldes-mongodb<br>\
    image: mongo:6.0.4<br>

    ports:<br>\
      - 27017:27017<br>

    networks:<br>\
      - ldes<br>

  ldes-cli:<br>\
    image: <br>ghcr.io/informatievlaanderen/ldes-cli:20230222T0959<br>\
    container_name: quick-start_ldes-client-cli<br>\
    command: "--url <br>http://localhost:8080/sample/by-page --input-format text/turtle"<br>\
    profiles:<br>\
      - delayed-started<br>\
    network_mode: service:ldes-server

networks:<br>\
  ldes:<br>\
    name: quick_start_network<br>

```

2. Please run\
   curl -X POST http://localhost:8080/sample -H "Content-Type: application/ttl" -d '@sample.ttl to post the sample.ttlto the [LDES Server](https://github.com/Informatievlaanderen/VSDS-LDESServer4J)

3. **Now!** [LDES Server](https://github.com/Informatievlaanderen/VSDS-LDESServer4J) has [Pagination fragmentation](https://github.com/Informatievlaanderen/VSDS-LDESServer4J/tree/main/ldes-fragmentisers/ldes-fragmentisers-pagination) -ed your first [LDES](https://semiceu.github.io/LinkedDataEventStreams/) data [sample.ttl](https://github.com/xdxxxdx/GetStarted_VSDS/blob/main/sample.ttl). Please use your preferred browser to reach <http://localhost:8080/sample> to have a look.

_The result should be as follow:_

```

@prefix ldes:   <[https://w3id.org/ldes#](https://w3id.org/ldes)> .<br>\
@prefix rdf:    <br><[http://www.w3.org/1999/02/22-rdf-syntax-ns#](http://www.w3.org/1999/02/22-rdf-syntax-ns)> .<br>\
@prefix sample: <br><http://localhost:8080/sample/> .<br>\
@prefix tree:   <https://w3id.org/tree#> .<br>

<http://localhost:8080/sample><br>

        rdf:type   ldes:EventStream ;<br>\
        tree:view  sample:by-page .<br>

sample:by-page  rdf:type  tree:Node .<br>

```

Follow the `tree:node` <http://localhost:8080/sample/by-page?pageNumber=1>, you could reach the first page of your data set.


### Synchronise your LDES dataset using [LDES Client](https://github.com/orgs/Informatievlaanderen/packages/container/package/ldes-cli)

- Please run docker compose up ldes-cli -d to start [LDES Client](https://github.com/orgs/Informatievlaanderen/packages/container/package/ldes-cli) docker container.

- The posted LDES stream: <http://localhost:8080/sample/by-time>will be followed and LDES members will be outputted to the console (only once) of the ldes-cli docker container. As follow:


```
2023-03-12 21:48:00 [pool-1-thread-1] INFO be.vlaanderen.informatievlaanderen.ldes.client.cli.services.FragmentProcessor - Fragment http://localhost:8080/sample/by-page has 0 member(s)
   2023-03-12 21:48:00 [pool-1-thread-1] INFO be.vlaanderen.informatievlaanderen.ldes.client.cli.services.FragmentProcessor - Fragment http://localhost:8080/sample/by-page?pageNumber=1 has 1 member(s)
   2023-03-12 21:48:01 _:B1e68397745bc55c40e0c5eadca8646e0 <https://schema.org/unitCode> <https://data.vmm.be/id/HP> .
   2023-03-12 21:48:01 _:B1e68397745bc55c40e0c5eadca8646e0 <https://schema.org/value> "1442"^^<http://www.w3.org/2001/XMLSchema#integer> .
   2023-03-12 21:48:01 <urn:ngsi-ld:WaterQualityObserved:woq:1/2023-03-12T18:31:17.003Z> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <https://www.w3.org/TR/vocab-ssn-ext/#sosa:ObservationCollection> .
   2023-03-12 21:48:01 <urn:ngsi-ld:WaterQualityObserved:woq:1/2023-03-12T18:31:17.003Z> <http://def.isotc211.org/iso19156/2011/SamplingFeature#SF_SamplingFeatureCollection.member> <https://data.vmm.be/id/loc-00019-33> .
   2023-03-12 21:48:01 <urn:ngsi-ld:WaterQualityObserved:woq:1/2023-03-12T18:31:17.003Z> <http://def.isotc211.org/iso19156/2011/SamplingFeature#SF_SamplingFeatureCollection.member> _:Bf2d869f7dc0b939c5b6016ac7d1c451c .
   2023-03-12 21:48:01 <urn:ngsi-ld:WaterQualityObserved:woq:1/2023-03-12T18:31:17.003Z> <http://def.isotc211.org/iso19156/2011/SamplingFeature#SF_SamplingFeatureCollection.member> _:B505977c4390a72ab0da767101451604d .
   2023-03-12 21:48:01 <urn:ngsi-ld:WaterQualityObserved:woq:1/2023-03-12T18:31:17.003Z> <http://def.isotc211.org/iso19156/2011/SamplingFeature#SF_SamplingFeatureCollection.member> _:Bdc3ef60306183fa17844d907314f40ee .
   2023-03-12 21:48:01 <urn:ngsi-ld:WaterQualityObserved:woq:1/2023-03-12T18:31:17.003Z> <http://purl.org/dc/terms/isVersionOf> <urn:ngsi-ld:WaterQualityObserved:woq:1> .
   2023-03-12 21:48:01 <urn:ngsi-ld:WaterQualityObserved:woq:1/2023-03-12T18:31:17.003Z> <http://www.w3.org/ns/prov#generatedAtTime> "2023-03-12T18:31:17.003Z"^^<http://www.w3.org/2001/XMLSchema#dateTime> .
   2023-03-12 21:48:01 <urn:ngsi-ld:WaterQualityObserved:woq:1/2023-03-12T18:31:17.003Z> <http://www.w3.org/ns/sosa/hasFeatureOfInterest> "spt-00035-79" .
   2023-03-12 21:48:01 _:B1e0a50c94effea51ea151d334d1d70a4 <https://schema.org/unitCode> <https://data.vmm.be/id/CEL> .
   2023-03-12 21:48:01 _:B1e0a50c94effea51ea151d334d1d70a4 <https://schema.org/value> "2.043000e+1"^^<http://www.w3.org/2001/XMLSchema#double> .
   2023-03-12 21:48:01 _:Bdc3ef60306183fa17844d907314f40ee <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://def.isotc211.org/iso19156/2011/Measurement#OM_Measurement> .
   2023-03-12 21:48:01 _:Bdc3ef60306183fa17844d907314f40ee <http://def.isotc211.org/iso19156/2011/Observation#OM_Observation.featureOfInterest> <https://data.vmm.be/id/spt-00035-79> .
   2023-03-12 21:48:01 _:Bdc3ef60306183fa17844d907314f40ee <http://def.isotc211.org/iso19156/2011/Observation#OM_Observation.observedProperty> <https://data.vmm.be/concept/waterkwaliteitparameter/temperatuur> .
   2023-03-12 21:48:01 _:Bdc3ef60306183fa17844d907314f40ee <http://def.isotc211.org/iso19156/2011/Observation#OM_Observation.phenomenonTime> "2023-03-12T18:31:17.003Z"^^<http://www.w3.org/2001/XMLSchema#datetime> .
   2023-03-12 21:48:01 _:Bdc3ef60306183fa17844d907314f40ee <http://def.isotc211.org/iso19156/2011/Observation#OM_Observation.result> _:B0d2c29f506d81bb4b3ae1170f6b7ae96 .
   2023-03-12 21:48:01 _:Bdc3ef60306183fa17844d907314f40ee <http://www.w3.org/ns/sosa/madeBySensor> <urn:ngsi-v2:cot-imec-be:Device:imec-iow-UR5gEycRuaafxnhvjd9jnU> .
   2023-03-12 21:48:01 _:B505977c4390a72ab0da767101451604d <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://def.isotc211.org/iso19156/2011/Measurement#OM_Measurement> .
   2023-03-12 21:48:01 _:B505977c4390a72ab0da767101451604d <http://def.isotc211.org/iso19156/2011/Observation#OM_Observation.featureOfInterest> <https://data.vmm.be/id/spt-00035-79> .
   2023-03-12 21:48:01 _:B505977c4390a72ab0da767101451604d <http://def.isotc211.org/iso19156/2011/Observation#OM_Observation.observedProperty> <https://data.vmm.be/concept/observatieparameter/hydrostatische-druk> .
   2023-03-12 21:48:01 _:B505977c4390a72ab0da767101451604d <http://def.isotc211.org/iso19156/2011/Observation#OM_Observation.phenomenonTime> "2023-03-12T18:31:17.003Z"^^<http://www.w3.org/2001/XMLSchema#datetime> .
   2023-03-12 21:48:01 _:B505977c4390a72ab0da767101451604d <http://def.isotc211.org/iso19156/2011/Observation#OM_Observation.result> _:B8e72081a21073160d62ed708aed78a23 .
   2023-03-12 21:48:01 _:B505977c4390a72ab0da767101451604d <http://www.w3.org/ns/sosa/madeBySensor> <urn:ngsi-v2:cot-imec-be:Device:imec-iow-UR5gEycRuaafxnhvjd9jnU> .
   2023-03-12 21:48:01 _:B8e72081a21073160d62ed708aed78a23 <http://def.isotc211.org/iso19103/2005/UnitsOfMeasure#Measure.value> _:B1e68397745bc55c40e0c5eadca8646e0 .
   2023-03-12 21:48:01 _:B22abd0fb7f46cf69b7367d7743860b0a <http://def.isotc211.org/iso19103/2005/UnitsOfMeasure#Measure.value> _:B66dbb43f4c55b6a44e931289145be554 .
   2023-03-12 21:48:01 <https://data.vmm.be/id/loc-00019-33> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://def.isotc211.org/iso19156/2011/SpatialSamplingFeature#SF_SpatialSamplingFeature> .
   2023-03-12 21:48:01 <http://localhost:8080/sample> <https://w3id.org/tree#member> <urn:ngsi-ld:WaterQualityObserved:woq:1/2023-03-12T18:31:17.003Z> .
   2023-03-12 21:48:01 _:B0d2c29f506d81bb4b3ae1170f6b7ae96 <http://def.isotc211.org/iso19103/2005/UnitsOfMeasure#Measure.value> _:B1e0a50c94effea51ea151d334d1d70a4 .
   2023-03-12 21:48:01 _:Bf2d869f7dc0b939c5b6016ac7d1c451c <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://def.isotc211.org/iso19156/2011/Measurement#OM_Measurement> .
   2023-03-12 21:48:01 _:Bf2d869f7dc0b939c5b6016ac7d1c451c <http://def.isotc211.org/iso19156/2011/Observation#OM_Observation.featureOfInterest> <https://data.vmm.be/id/spt-00035-79> .
   2023-03-12 21:48:01 _:Bf2d869f7dc0b939c5b6016ac7d1c451c <http://def.isotc211.org/iso19156/2011/Observation#OM_Observation.observedProperty> <https://data.vmm.be/concept/waterkwaliteitparameter/conductiviteit> .
   2023-03-12 21:48:01 _:Bf2d869f7dc0b939c5b6016ac7d1c451c <http://def.isotc211.org/iso19156/2011/Observation#OM_Observation.phenomenonTime> "2023-03-12T18:31:17.003Z"^^<http://www.w3.org/2001/XMLSchema#datetime> .
   2023-03-12 21:48:01 _:Bf2d869f7dc0b939c5b6016ac7d1c451c <http://def.isotc211.org/iso19156/2011/Observation#OM_Observation.result> _:B22abd0fb7f46cf69b7367d7743860b0a .
   2023-03-12 21:48:01 _:Bf2d869f7dc0b939c5b6016ac7d1c451c <http://www.w3.org/ns/sosa/madeBySensor> <urn:ngsi-v2:cot-imec-be:Device:imec-iow-UR5gEycRuaafxnhvjd9jnU> .
   2023-03-12 21:48:01 _:B66dbb43f4c55b6a44e931289145be554 <https://schema.org/unitCode> <https://data.vmm.be/id/G42> .
   2023-03-12 21:48:01 _:B66dbb43f4c55b6a44e931289145be554 <https://schema.org/value> "6150"^^<http://www.w3.org/2001/XMLSchema#integer> .
```


### Tear down the infrastructure and remove the volumes


Within the working directory, please run

 docker compose down -v




## Example 2

```note
***Please Note!***

*This quick start try-out is ONLY made for making *[Pagination fragmentation](https://github.com/Informatievlaanderen/VSDS-LDESServer4J/tree/main/ldes-fragmentisers/ldes-fragmentisers-pagination)* for the self-generated data type to *[LDES format](https://semiceu.github.io/LinkedDataEventStreams/)*. To support other fragments, data types, or other features, e.g., retention, caching, etc., please consult *[LDES Server Manual](https://github.com/Informatievlaanderen/VSDS-LDESServer4J)* for the configurations.*

[Vlaamse Smart Data Space](https://www.vlaanderen.be/digitaal-vlaanderen/onze-oplossingen/vlaamse-smart-data-space)* projects also provide methods for transforming data to *[LDES format](https://semiceu.github.io/LinkedDataEventStreams/)*, e.g. from *[NGSI-V2](https://vloca-kennishub.vlaanderen.be/NGSI-v2)* to *[NGSI-LD](https://en.wikipedia.org/wiki/NGSI-LD)*, from NGSI to *[OSLO](https://www.vlaanderen.be/digitaal-vlaanderen/onze-oplossingen/oslo)* Model, from *[NGSI-V2](https://vloca-kennishub.vlaanderen.be/NGSI-v2)* to *[LDES (LinkedDataEventStreams)](https://semiceu.github.io/LinkedDataEventStreams/)

```


In this short example below, we show you how the components of the VSDS can be used to link multiple data streams across domains and systems using LDES. The whole story can be read here: <https://medium.com/p/e1aab3090b40/edit>

### Before starting


1.  [Docker](https://docker.com/) has been installed on your device.
2.  Local Ports 8443 and 7200 are accessible.

### Start your LDES workbench


1.  Create a local [docker-compose.yml](https://stackedit.io/docker-compose.yml) file with the following content, or you could use the current good-to-go [GetStarted GRAR project](https://github.com/samuvack/ldes-grar):

```
version: ‘3’
volumes:
  nifi-database-repository:
  nifi-flowfile-repository:
  nifi-content-repository:
  nifi-provenance-repository:
  nifi-state:
  nifi-logs:
  nifi-nars:
  nifi-conf:
networks:
  front-tier:
    driver: bridge
  back-tier:
    driver: bridge
services:
  nifi:
    image: ghcr.io/informatievlaanderen/ldes-workbench-nifi:20230214T123440 #20221109T103745
    container_name: nifi-graph
    restart: unless-stopped
    ports:
      # HTTPS
      - 8443:8443/tcp
    volumes:
      - ./nifi-extensions:/opt/nifi/nifi-current/extensions
    environment:
      - NIFI_UI_PORT=8443
      - SINGLE_USER_CREDENTIALS_USERNAME=admin
      - SINGLE_USER_CREDENTIALS_PASSWORD=admin123456789
      - NIFI_WORKFLOW_LISTEN_PORT=9005
      - NIFI_JVM_HEAP_INIT=8g
      - NIFI_JVM_HEAP_MAX=8g
  graphdb:
    image: ontotext/graphdb:10.0.2
    container_name: graphdb3
    ports:
      # HTTP
      - 7200:7200
```

To begin, the command for launching GraphDb and Apache NiFi within a docker container is as follows:

```
docker-compose up --build
```
Apache Nifi runs on port 8443:8443/tcp :[https://localhost:8443](https://localhost:8443/)\
GraphDB runs on port 7200 :7200/tcp : [https://localhost:7200](https://localhost:7200/)[https://localhost:7200](https://localhost:7200/)

A data flow can be built up by dragging in building blocks in the graphical user interface of Apache NiFi. This docker container contains all the components out the latest version of the LDES workbench (e.g., latest LDES Client, LDES server, etc.). Every data stream can be transformed into a Linked Data Event Stream with a conversion data flow.

### Publishing your first [LDES](https://semiceu.github.io/LinkedDataEventStreams/)


In this example, we start from data streams outputted via Apache Kafka.

![](file:///C:/Users/samue/AppData/Local/Temp/msohtmlclip1/01/clip_image001.png)

A data flow is configured in Apache Nifi that converts these Kafka topics to an LDES server. The LDES server is a configurable component used to ingest, store, transform, and (re-)publish a Linked Data Event Stream. For more information about an LDES server, go to LDES server.

Because we are dealing with 'state' objects, we need to convert these objects into version objects. The data flow below converts the Apache Kafka topics into JSON members. Afterwards, the spatial reference system is changed to the local coordinate system of Flanders (Lambert 72)*.*

1. Go to localhost:8433/

2. Login with credentials

3. Open data flow by importing Apache NiFi configuration file

4. Kafka topic

5. Start Apachi NiFi data flow

![](file:///C:/Users/samue/AppData/Local/Temp/msohtmlclip1/01/clip_image003.png)

Now that the Kafka topics are converted into a version object, the LDES server can publish these members into LDES members.

```
@prefix conceptscheme: <https://data.vlaanderen.be/id/conceptscheme/> .
@prefix generiek:      <https://data.vlaanderen.be/ns/generiek#> .
@prefix geosparql:     <http://www.opengis.net/ont/geosparql#> .
@prefix ldes:          <https://w3id.org/ldes#> .
@prefix locn:          <https://www.w3.org/ns/locn#> .
@prefix prov:          <http://www.w3.org/ns/prov#> .
@prefix rdf:           <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix relation:      <http://www.iana.org/assignments/relation/> .
@prefix terms:         <http://purl.org/dc/terms/> .
@prefix tree:          <https://w3id.org/tree#> .
https://data.vlaanderen.be/id/adres/1658513
        relation:self  "https://api.basisregisters.staging-vlaanderen.be/v2/adressen/1658513" .
[ https://data.vlaanderen.be/ns/gebouw#bestaatUit
          <https://data.vlaanderen.be/id/gebouweenheid/17631816/2022-06-27T16:23:49+02:00> ;
  generiek:lokaleIdentificator  "17631524"] .
[ https://data.vlaanderen.be/ns/gebouw#bestaatUit
<https://data.vlaanderen.be/id/gebouweenheid/6311633/2022-06-27T16:23:48+02:00> ;  generiek:lokaleIdentificator  "6310652"] .
https://data.vlaanderen.be/id/gebouweenheid/14539518/2022-06-27T16:23:32+02:00
   rdf:type                      <https://data.vlaanderen.be/ns/gebouw#Gebouweenheid> ;
        terms:isVersionOf             <https://data.vlaanderen.be/id/gebouweenheid/14539518> ;        prov:generatedAtTime          "2022-06-27T16:23:32+02:00"^^<http://www.w3.org/2001/XMLSchema#dateTime> ;
<https://data.vlaanderen.be/ns/gebouw#Gebouweenheid.adres>                <https://data.vlaanderen.be/id/adres/5162364> ;        <https://data.vlaanderen.be/ns/gebouw#Gebouweenheid.geometrie>
                [ conceptscheme:geometriemethode  conceptscheme:aangeduidDoorBeheerder ;                  locn:geometry                   [ geosparql:asGML                                    "<gml:Point srsName=\"http://www.opengis.net/def/crs/EPSG/0/31370\" xmlns:gml=\"http://www.opengis.net/gml/3.2\"><gml:pos>200647.71 210393.46</gml:pos></gml:Point>"^^geosparql:gmlLiteral ]                ] ;        <https://data.vlaanderen.be/ns/gebouw#Gebouweenheid.status>
                conceptscheme:gerealiseerd ;       <https://data.vlaanderen.be/ns/gebouw#functie>                conceptscheme:nietGekend ;
        generiek:lokaleIdentificator  "14539518" ;        generiek:naamruimte           "https://data.vlaanderen.be/id/gebouweenheid" ;        generiek:versieIdentificator  "2022-06-27T16:23:32+02:00" .
<https://data.vlaanderen.be/id/adres/2213034>        relation:self  "https://api.basisregisters.staging-vlaanderen.be/v2/adressen/2213034" .
```


### Consuming an LDES


For this purpose, the three Linked Data Event streams are stored in a [GraphDB](https://graphdb.ontotext.com/) to facilitate efficient and effective data consumption. GraphDB supports complex semantic queries and inference, making discovering meaningful relationships between different data sources possible.

![Afbeelding met grafiek

Automatisch gegenereerde beschrijving](file:///C:/Users/samue/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

1. Start Apache NiFi on localhost:8433

2. Import data flow by importing [Apache NiFi configuration file](https://github.com/samuvack/ldes-grar/blob/main/NiFi_Flow.json)

3. Add [LDES endpoint](https://grar.smartdataspace.dev-vlaanderen.be/building-units/by-page) in the LDES client

4. Start Apache NiFi data flow

![](file:///C:/Users/samue/AppData/Local/Temp/msohtmlclip1/01/clip_image004.png)

5. Open GraphDB on localhost:7200/sparql

6. Run a semantic query with SPARQL

![](file:///C:/Users/samue/AppData/Local/Temp/msohtmlclip1/01/clip_image006.png)

Below you find an example of a SPARQL query. This query returns triples with information about the parcel LDES.

```
PREFIX adres: https://data.vlaanderen.be/id/adres/
PREFIX gebouw: https://data.vlaanderen.be/id/?gebouweenheid/
PREFIX gebouweenheid: https://data.vlaanderen.be/ns/gebouw#
PREFIX prov: http://www.w3.org/ns/prov#
PREFIX generiek: https://data.vlaanderen.be/ns/generiek#
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
construct {
?gebouweenheid prov:generatedAtTime ?generatedAtTime .
?gebouweenheid rdf:type ?type .?gebouweenheid gebouweenheid:Gebouweenheid.geometrie ?geometrie .
?gebouweenheid gebouweenheid:Gebouweenheid.status ?status .
?gebouweenheid gebouweenheid:functie ?functie .
?gebouweenheid generiek:lokaleIdentificator ?lokaleIdentificator .
?gebouweenheid generiek:naamruimte ?naamruimte .
?gebouweenheid generiek:versieIdentificator ?versieIdentificator .  
} where { 
	?gebouweenheid <https://data.vlaanderen.be/ns/gebouw#Gebouweenheid.adres> <https://data.vlaanderen.be/id/adres/1864311> .
    OPTIONAL{
    ?gebouweenheid prov:generatedAtTime ?generatedAtTime .
	?gebouweenheid rdf:type ?type .
	?gebouweenheid gebouweenheid:Gebouweenheid.geometrie ?geometrie .
	?gebouweenheid gebouweenheid:Gebouweenheid.status ?status .
	?gebouweenheid gebouweenheid:functie ?functie .
	?gebouweenheid generiek:lokaleIdentificator ?lokaleIdentificator .
	?gebouweenheid generiek:naamruimte ?naamruimte .
	?gebouweenheid generiek:versieIdentificator ?versieIdentificator .  
}}
```
This SPARQL query returns multiple triples with information about building units. These triples can then be converted into an RDF file.

The RDF file should be as follow:
```
@prefix ns1: <https://data.vlaanderen.be/ns/generiek#> .
@prefix ns2: <https://data.vlaanderen.be/ns/gebouw#> .
@prefix ns3: <http://www.w3.org/ns/prov#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
<https://data.vlaanderen.be/id/gebouweenheid/14635685> a ns2:Gebouweenheid ;
    ns3:generatedAtTime "2022-06-27T20:51:02+02:00"^^xsd:dateTime ;    ns2:Gebouweenheid.geometrie [ ] ;    ns2:Gebouweenheid.status <https://data.vlaanderen.be/id/conceptscheme/gerealiseerd> ;    
ns2:functie <https://data.vlaanderen.be/id/conceptscheme/nietGekend> ;    ns1:lokaleIdentificator "14635685" ;
ns1:naamruimte "https://data.vlaanderen.be/id/gebouweenheid" ;    ns1:versieIdentificator "2022-06-27T20:51:02+02:00" .
```


