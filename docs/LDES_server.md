---
sort: 4
---




# LDES SERVER






The Linked Data Event Stream (LDES) [server](https://github.com/Informatievlaanderen/VSDS-LDESServer4J) is a configurable component that can be used to ingest, store, and (re-)publish an LDES. The LDES server was built in the context of the VSDS project to exchange Open Data easily.

<p align="center"><img src="/VSDS-Tech-Docs/images/LDES%20server.png" width="60%" text-align="center"></p>

To help data publishers make their datasets available as LDES, an open-source LDES server was developed in the context of the [VSDS project](https://www.vlaanderen.be/vlaamse-smart-data-space-portaal). The server can be configured to meet the organisation's specific needs. Functionalities include **retention policy**, **fragmentation** and **pagination**  for managing and processing large amounts of data more efficiently and ensuring the efficient use of storage.

![](../images/scalableApplications.png)

```note
The LDES server is available as on open-source building block on [GitHub](https://github.com/Informatievlaanderen/VSDS-LDESServer4J)
```

# Event Stream

<p align="center"><img src="/VSDS-Tech-Docs/images/Get_LDES.png" text-align="center"></p>






<p align="center"><img src="/VSDS-Tech-Docs/images/Put_LDES.png" text-align="center"></p>






<p align="center"><img src="/VSDS-Tech-Docs/images/Get_collection.png" text-align="center"></p>






<p align="center"><img src="/VSDS-Tech-Docs/images/Delete_Ldes.png" text-align="center"></p>





# Views

<p align="center"><img src="/VSDS-Tech-Docs/images/Get_collection.png" text-align="center"></p>





<p align="center"><img src="/VSDS-Tech-Docs/images/Put_view.png" text-align="center"></p>




<p align="center"><img src="/VSDS-Tech-Docs/images/Get_config_view.png" text-align="center"></p>




<p align="center"><img src="/VSDS-Tech-Docs/images/Delete_view.png" text-align="center"></p>



# Shapes


<p align="center"><img src="/VSDS-Tech-Docs/images/Get_shape.png" text-align="center"></p>


<p align="center"><img src="/VSDS-Tech-Docs/images/Put_shape.png" text-align="center"></p>


# Snapshots

<p align="center"><img src="/VSDS-Tech-Docs/images/Post_snapshot.png" text-align="center"></p>






## Ingesting sources (HTTP in)

The LDES server is able to receive data via HTTP ingestion. Specifically, the server expects a single object (member) to be sent as input via a POST request. If the dataset still contains state objects, each of these must first be converted to a version object before being ingested in the server. This essential step ensures the ingested objects comply with the [LDES definition](/docs/Specification.md#what-is-a-linked-data-event-stream).

Once the objects in the dataset are LDES-compliant members (whether or not after conversion to a version object) and the LDES member has been added to the LDES server, the server can effortlessly publish the LDES member as part of the LDES.

More information on the HTTP ingestion can be found [here](https://github.com/Informatievlaanderen/VSDS-LDESServer4J#example-http-ingest-fetch-configuration).

Example HTTP Ingest-Fetch Configuration:

```yaml
server.port: { http-port }
ldes:
  collection-name: { short name of the collection, cannot contain characters that are used for url interpretation, e.g.’ $’, ‘=’ or ‘&’}
  host-name: { hostname of LDES Server }
  member-type: { Defines which syntax type is used to define the member id e.g. “https://data.vlaanderen.be/ns/mobiliteit#Mobiliteitshinder”}
  timestamp-path: { SHACL property path to the timestamp when the version object entered the event stream. }
  version-of: { SHACL property path to the non-versioned identifier of the entity. }
  validation:
    shape: { URI to defined shape }
    enabled: { Enables/Disables shacl validation on ingested members }
rest:
  max-age: { time in seconds that a mutable fragment can be considered up-to-date, default when omitted: 60 }
  max-age-immutable: { time in seconds that an immutable fragment should not be refreshed, default when omitted: 604800 }
```

## SHACL

The LDES specification prescribes that each LDES must link to a SHACL shape, providing a machine-readable definition of the members in the collection. If a SHACL shape was provided on startup, the LDES server reads it before the ingestion process starts and the SHACL shape is used to validate the ingested members. Only valid members are ingested in the LDES server. When starting the server, it is possible to provide a SHACL shape via through an RDF file. At last, the SHACL shape is also published as part of the LDES on the Web.

```note
SHACL stands for Shapes Constraint Language and is used to define a set of constraints which are used to verify to conformity of RDF data with these constraints.
```

The SHACL shape specifies the expected properties of an LDES members and the constraints that must be followed to ensure the LDES member adheres to the expected structure and semantics. It defines properties such as required properties, allowed property values, and the data types expected for the properties.

For more information about the SHACL shape and its structure, go to [here](https://informatievlaanderen.github.io/VSDS-Tech-Docs/docs/Specification.html#shacl). More information on how to provide an RDF file, containing a SHACL shape, to the LDES server can be found [here](https://github.com/Informatievlaanderen/VSDS-LDESServer4J#example-serving-static-content).

## Fragmentation

To reduce the volume of data that consumers need to replicate or to speed up certain queries, the LDES server can be configured to create several fragmentations. Fragmentations are similar to indexes in databases but then published on the Web. The RDF predicate on which the fragmentation must be applied is defined through configuration.

<p align="center"><img src="/VSDS-Tech-Docs/images/fragmentation.png" width="60%" text-align="center"></p>

The fragmenting of a Linked Data Event Stream (LDES) is a crucial technique for managing and processing large amounts of data more efficiently. There are three main methods of fragmentation: **geospatial**, **time-based**, and **substring** fragmentation.

<br>

<br>

---

### Partitioning

When applying partitioning, the LDES server will create fragments based on the order of arrival of the LDES member, and is a linear fragmentation. This fragmentation is considered the most basic and default fragmentation because it stands for the exact reason the LDES specification was created: replication and synchronising with a dataset. The members arriving on the LDES server are added to the first page, while the latest members are always included on the latest page.

The expected parameter to apply a partioning is a `member limit`, indicating the amount of members that can be added to each page before creating a new page.

```yaml
name: “pagination”
config:
  memberLimit: { Mandatory: member limit > 0 }
```

**Algorithm**

1. The fragment to which the member should be added is determined.
   - The currently open fragment is retrieved from the database.
   - If this fragment contains members equal to or exceeding the member limit or no fragment can be found, a new fragment is created instead.

2. If a new fragment is created, the following steps are taken.
   - The new fragment becomes the new open fragment and the previous fragment becomes immutable1.
   - This newly created fragment and the previous fragment are then linked with each other by 2 generic relationships1.
   - The pagenumber of the new fragment is determined based on the old fragment or is set to 1 in case of the first fragment.

1 In case of the first fragment, a previous fragment does not exist so these steps are skipped.

![](content/paginationAlgorithm.png)

**Example properties**

```yaml
name: "pagination"
config:
  memberLimit: 10
```

<br>

<br>

---

### Substring fragmentation

[Substring fragmentation](https://github.com/Informatievlaanderen/VSDS-LDESServer4J/tree/main/ldes-fragmentisers/ldes-fragmentisers-substring) involves dividing the data stream into smaller pieces based on specific substrings, or patterns, within the data.

Example of substring fragmentation configuration file

```yaml
name: “substring”
config:
  fragmenterProperty: { Defines which property will be used for bucketizing }
  memberLimit: { member limit > 0 }
```

**Example**

Example properties:

```yaml
name: "substring"
config:
  fragmenterProperty: "https://data.vlaanderen.be/ns/adres#volledigAdres"
  memberLimit: 10
```

With following example input:

```ttl
@prefix prov: <http://www.w3.org/ns/prov#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

<https://data.vlaanderen.be/id/adres/1781020/2023-02-15T10:14:36.002Z>
  <https://data.vlaanderen.be/ns/adres#isVerrijktMet> [ 
      <https://data.vlaanderen.be/ns/adres#volledigAdres> "Kazernestraat 15, 9160 Lokeren"@nl 
   ] ;
  prov:generatedAtTime "2023-02-15T10:14:36.002Z"^^xsd:dateTime ;
  a <https://data.vlaanderen.be/ns/adres#Adres> .
```

The selected object would be "Kazernestraat 15, 9160 Lokeren".

The bucket of substrings would be:

- K
- Ka
- Kaz
- ...
- Kazernestraat 15, 9160 Lokeren

If this is the first member of the collection it would be added to fragment 'k' and available at http://localhost:8080/addresses/by-name?substring=k and not in ka or any other substring fragment.

In a scenario where there are already 10 addresses starting with 'k' and only 2 with 'ka', it would be added to http://localhost:8080/addresses/by-name?substring=ka

```note
Note that this is all lowercase.
```

<br>

<br>

---

### Time-based fragmentation

[Time-based fragmentation](https://github.com/Informatievlaanderen/VSDS-LDESServer4J/tree/main/ldes-fragmentisers/ldes-fragmentisers-timebased) has not yet been implemented.

<p align="center"><img src="/VSDS-Tech-Docs/images/temporal.png" width="60%" text-align="center"></p>

Example of a time-based fragmentation configuration file

```yaml
name: “timebased”
config:
  memberLimit: { member limit > 0 }
```

**Algorithm**
This fragmentiser will create an initial fragment with the current timestamp when processing a member.
Members are added to the fragment until the member limit is reached. When the fragment member limit is reached, a
next fragment is created with a new current timestamp.

**Reasons for deprecating this fragmentiser:**

1. This fragmentiser follows the algorithm of pagination but without the semantics.
2. For a correct timebased fragmentation, members of the fragment should be checked and their value
   for a given property should be used to create the correct relations. This is not the case, and there is currently no demand to have this implemented.

<br>

<br>

---

### Geospatial fragmentation

Consider the scenario where the address registry is published as an LDES that using partitioning. In such a case, data consumers are required to replicate the entire linear set of fragments, despite only being interested in a smaller subset of the dataset. For instance, the city of Brussels may only require addresses within its geographical region and is not interested in other addresses. However, with the partitioned LDES, they would need to iterate through all the fragments and filter the LDES members (address version objects) on the client-side. By utilising geospatial fragmentation, the data can be divided into smaller pieces (tiles) based on geographical location. This facilitates filtering on the fragment level (tiles) and allows for processing and analysis of data within specific geospatial tiles.

<p align="center"><img src="/VSDS-Tech-Docs/images/geospatial.png" width="60%" text-align="center"></p>

The geospatial fragmentation supported by the LDES server is based on the ["Slippy Maps" algorithm](https://wiki.openstreetmap.org/wiki/Slippy_map). The fragmentation expects a `zoom level` parameter which is used by the algorithm to divide the "world" into tiles. The number of tiles if 2^2n^ (where n = zoom level). The second expected parameter is an `RDF predicate`, indicating on which property of the LDES member the fragmentation should be applied. More information about the algorithm used to apply a geospatial fragmentation can be found [here](https://github.com/Informatievlaanderen/VSDS-LDESServer4J/tree/main/ldes-fragmentisers/ldes-fragmentisers-geospatial).

The required configuration for this fragmentation is:

1. RDF predicate on which the fragmentation should be based
2. Zoom level

Example of geospatial fragmentation configuration file

```yaml
name: “geospatial”
config:
  maxZoomLevel: { Required zoom level }
  fragmenterProperty: { Defines which property will be used for bucketizing }
```

**Algorithm**

1. The fragmentationObjects of the member are determined
   - We filter the RDF statements where the predicate matches the `fragmenterProperty`
   - If an optional regex is provided through the `fragmenterSubjectFilter` property, we filter on subjects that match this regex.
   - We select all the object that pass the above filters.

2. A bucket of tiles is created using the coordinates and provided zoomLevel. [This is done using the Slippy Map algorithm.](https://wiki.openstreetmap.org/wiki/Slippy_map)
3. The tiles are iterated. The member is added to every tile, or sub-fragmentations of these tiles1. Taking into account:
   - A new fragment is created if no fragment exists for the given tile.
   - There is no `memberLimit` or max size for a fragment. They do not become immutable.
   - The member is added to every related fragment

1 If the geospatial fragmentation is not the lowest fragmentation level, the member is not added to the tile but to a subfragment on this tile. This case is included in the [example below](#when-we-have-a-timebased-sub-fragmentation-below-geospatial-fragmentation).

![](content/geospatial_algorithm.drawio.png)

**Example**

Example properties:

```yaml
name: "geospatial"
config:
  maxZoomLevel: 15
  fragmenterProperty: "http://www.opengis.net/ont/geosparql#asWKT"
```

With following example input:

```ttl
@prefix dc: <http://purl.org/dc/terms/> .
@prefix ns0: <http://semweb.mmlab.be/ns/linkedconnections#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix ns1: <http://vocab.gtfs.org/terms#> .
@prefix prov: <http://www.w3.org/ns/prov#> .
@prefix ns2: <http://www.opengis.net/ont/geosparql#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix geo: <http://www.w3.org/2003/01/geo/wgs84_pos#> .

<http://njh.me/original-id#2022-09-28T17:11:28.520Z>
  dc:isVersionOf <http://njh.me/original-id> ;
  ns0:arrivalStop <http://example.org/stops/402161> ;
  ns0:arrivalTime "2022-09-28T07:14:00.000Z"^^xsd:dateTime ;
  ns0:departureStop <http://example.org/stops/402303> ;
  ns0:departureTime "2022-09-28T07:09:00.000Z"^^xsd:dateTime ;
  ns1:dropOffType ns1:Regular ;
  ns1:pickupType ns1:Regular ;
  ns1:route <http://example.org/routes/Hasselt_-_Genk> ;
  ns1:trip <http://example.org/trips/Hasselt_-_Genk/Genk_-_Hasselt/20220928T0909> ;
  a ns0:Connection ;
  prov:generatedAtTime "2022-09-28T17:11:28.520Z"^^xsd:dateTime .

<http://example.org/stops/402161>
  ns2:asWKT "POINT (5.47236 50.9642)"^^ns2:wktLiteral ;
  a ns1:Stop ;
  rdfs:label "Genk Brug" ;
  geo:lat 5.096420e+1 ;
  geo:long 5.472360e+0 .

<http://example.org/stops/402303>
  ns2:asWKT "POINT (5.49661 50.9667)"^^ns2:wktLiteral ;
  a ns1:Stop ;
  rdfs:label "Genk Station perron 11" ;
  geo:lat 5.096670e+1 ;
  geo:long 5.496610e+0 .
```

The selected objects would be

`"POINT (5.47236 50.9642)"^^ns2:wktLiteral` and `"POINT (5.49661 50.9667)"^^ns2:wktLiteral`

When we convert these [coordinates to tiles](https://wiki.openstreetmap.org/wiki/Slippy_map_tilenames#Lon..2Flat._to_tile_numbers_2), the bucket of tiles would be:

- "15/16884/10974"
- "15/16882/10975"

When geospatial fragmentation is the lowest level

After ingestion the member will be part of the following two fragments

- http://localhost:8080/addresses/by-zone?tile=15/16884/10974
- http://localhost:8080/addresses/by-zone?tile=15/16882/10975

When we have a timebased sub-fragmentation below geospatial fragmentation

After ingestion the member will be part of the following two fragments

- http://localhost:8080/addresses/by-zone-and-time?tile=15/16884/10974&generatedAtTime=2023-02-15T10:14:28.262Z
- http://localhost:8080/addresses/by-zone-and-time?tile=15/16882/10975&generatedAtTime=2023-02-15T10:14:28.262Z

Note that the `generatedAtTime=2023-02-15T10:14:28.262Z` is an example, this can be any other fragmentation.

<br>

<br>

---

#### Combining geospatial fragmentation and partioning

The LDES server typically adds an LDES member to the "lowest" possible fragment, which in the case of geospatial fragmentation, would be the fragment representing the corresponding geospatial tile. However, some fragments/tiles may have many members, while others may have none. In the worst-case scenario, all members may be added to one geospatial tile/fragment, leading to an enormous fragment. Combining geospatial fragmentation with partitioning can be a useful approach to mitigate this issue. Then partitioning is applied within every geospatial tile, resulting in a set of linear fragments for every geospatial tile. Doing so, all members can still end up in the same geospatial tile, but now clients have a set of linear fragments to iterate through instead of one enormous fragment. More detailed information is available in the [example below](#when-we-have-a-timebased-sub-fragmentation-below-geospatial-fragmentation).

![](content/geospatial_algorithm.drawio.png)

#### Example → TODO: create more easy example (e.g. Addresses)

---

## Retention policy

A [retention policy](https://github.com/Informatievlaanderen/VSDS-LDESServer4J#example-retention) determines how long data will be kept and stored. Its purpose is to ensure the efficient use of storage resources by controlling data growth over time. Setting a retention policy per view to minimise storage fill-up is possible.

<p align="center"><img src="/VSDS-Tech-Docs/images/retention_policy.png" width="60%" text-align="center"></p>

Implementing a retention policy helps organisations maintain control over their data growth and ensure that storage resources are used optimally. The policy specifies the maximum duration that data should be kept. Currently, the only retention policy supported is time-based, which can be configured using the [ISO 8601](https://tc39.es/proposal-temporal/docs/duration.html) duration format. This time-based policy ensures that data is automatically deleted after a specified period, freeing up valuable storage space for new data.

```yaml
views:
  - name: “firstView”
    retentionPolicies:
      - name: “timebased”
        config:
          duration: “PT5M”
```

duration:  "PT5M"

As an example, the time-based retention configuration example above is set up to ensure that data is automatically deleted after 5 minutes (PT5M).

## Hosting the LDES stream SHACL shape

SHACL (Shapes Constraint Language) is a language used to validate RDF graphs against a set of conditions provided as shapes and other constructs in an RDF graph. The LDES Server facilitates hosting a SHACL shape describing the members in the LDES. Through configuration, it is possible to [reference an existing SHACL shape](https://github.com/Informatievlaanderen/VSDS-LDESServer4J#example-http-ingest-fetch-configuration) via an URL or to provide a [static file](https://github.com/Informatievlaanderen/VSDS-LDESServer4J#example-serving-static-content) with an RDF description of the SHACL shape.

## Hosting DCAT metadata

DCAT is a standardised RDF vocabulary to describe data catalogues on the Web, allowing easy interoperability between catalogues. Using a standard schema, DCAT enhances discoverability and facilitates federated search across multiple catalogues.

The LDES server facilitates hosting DCAT metadata when publishing an LDES. Through configuration, as with the SHACL shape, it is possible to reference an existing DCAT via an URI or to provide a static file containing an RDF description of the DCAT.
More information on configuring DCAT on the LDES Server can be found [here](https://github.com/Informatievlaanderen/VSDS-LDESServer4J#example-serving-dcat-metadata).

## Setup of the LDES Server

A detailed description on how to setup the LDES Server can be found in the [README.md](https://github.com/Informatievlaanderen/VSDS-LDESServer4J#ldes-server) of the LDES Server GitHub repository.
