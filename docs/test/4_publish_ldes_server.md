---
sort: 8
---
# PUBLISH LDES


## LDES server

The Linked Data Event Stream (LDES) [server](https://github.com/Informatievlaanderen/VSDS-LDESServer4J) is a configurable component that can be used to ingest, store, and (re-)publish an LDES. The LDES server was built in the context of the VSDS project to exchange Open Data easily.

![](/images/LDES%20server.png)

The server can tailor its functionality to meet the organisation's specific needs. Functionalities include **retention policy**, **fragmentation** and **pagination**  for managing and processing large amounts of data more efficiently and ensuring the efficient use of storage. 

![](file:///C:/Users/samue/AppData/Local/Temp/msohtmlclip1/01/clip_image004.jpg)

### Ingesting sources (HTTP in)

The LDES server is able to receive data via HTTP ingestion. Specifically, the server expects an LDES-compliant member to be sent as input via a POST request. This member must be a version object of an entity that adheres to the LDES standard.

However, if the data stream is not yet LDES compliant, such as a state object data stream, it must first be converted to a version object. This essential step ensures data compatibility and adherence to the LDES standards.

Once the data stream is converted to a version object and the LDES-compliant member is received by the LDES server, the server can effortlessly publish the LDES member as part of the LDES.

When starting up the LDES server, the server checks if a SHACL shape (Shapes Constraint Language) is provided before proceeding with the ingestion process. A SHACL shape is used to validate members against predefined validation rules and to verify the conformity of RDF data with these rules.

The SHACL shape specifies the properties of an RDF resource and the rules that must be followed to ensure the data adheres to the expected structure and semantics. It defines properties such as required properties, allowed property values, and the data types expected for the properties.

For more information about the SHACL shape and its structure, **link:** **go to ****2.3**** SHACL Shape.**

### Example HTTP Ingest-Fetch Configuration:


```config
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

### Fragmentation

To reduce the amount of data consumers need to replicate or to speed up certain queries, the LDES server can be configured with several fragmentations. Fragmentations are similar to indexes in databases but then published on the Web. The RDF predicate on which the fragmentation must be applied is defined through configuration.

![fragmentation](/images/fragmentation.png)

The fragmenting of a Linked Data Event Stream (LDES) is a crucial technique for managing and processing large amounts of data more efficiently. There are three main methods of fragmentation: **geospatial**,** time-based**, and **substring** fragmentation.


**Partitioning**

When applying partitioning, the LDES server will create fragments based on the order of arrival of the LDES member. This fragmentation is considered to be the most basic and default fragmentation.\
The members that arrive first on the LDES server are added to the first page, while the latest members are always included on the latest page.



```
name: “pagination”
config:
  memberLimit: { Mandatory: member limit > 0 }
```

[**Substring fragmentation**](https://github.com/Informatievlaanderen/VSDS-LDESServer4J/tree/main/ldes-fragmentisers/ldes-fragmentisers-substring)

Substring fragmentation involves dividing the data stream into smaller pieces based on specific substrings, or patterns, within the data. 

Example of substring fragmentation configuration file

```
name: “substring”
config:
  fragmenterProperty: { Defines which property will be used for bucketizing }
  memberLimit: { member limit > 0 }
```
[**Time-based fragmentation**](https://github.com/Informatievlaanderen/VSDS-LDESServer4J/tree/main/ldes-fragmentisers/ldes-fragmentisers-timebased)

Time-based fragmentation has not yet been implemented.



![](/images/temporal.png)

Example of a time-based fragmentation configuration file



```
name: “timebased”
config:
  memberLimit: { member limit > 0 }
```

[**Geospatial fragmentation**](https://github.com/Informatievlaanderen/VSDS-LDESServer4J/tree/main/ldes-fragmentisers/ldes-fragmentisers-geospatial)

Geospatial fragmentation involves dividing the data stream into smaller pieces based on geographical information, allowing organisations to process and analyse data within specific geographic areas in real-time. 

![](/images/geospatial.png)

The geospatial fragmentation follows the "Slippy Maps" principle. A zoom level is set through configuration, and the "world" is divided into tiles based on this zoom level. The number of tiles is 2^2n^ (where n = zoom level). An RDF predicate must also be configured for this fragmentation, determining on which property of the LDES member the fragmentation should be applied ....

The required configuration for this fragmentation is:

1. RDF predicate on which the fragmentation should be based

2. Zoom level

Example of geospatial fragmentation configuration file
```
name: “geospatial”
config:
  maxZoomLevel: { Required zoom level }
  fragmenterProperty: { Defines which property will be used for bucketizing }
```

### Retention policy


A [retention policy](https://github.com/Informatievlaanderen/VSDS-LDESServer4J#example-retention) determines how long data will be kept and stored. Its purpose is to ensure the efficient use of storage resources by controlling data growth over time. Setting a retention policy per view to minimise storage fill-up is possible.

![Retention policy](/images/retention_policy.png)

Implementing a retention policy helps organisations maintain control over their data growth and ensure that storage resources are used optimally. The policy specifies the maximum duration that data should be kept. Currently, the only retention policy supported is time-based, which can be configured using the [ISO 8601](https://tc39.es/proposal-temporal/docs/duration.html) duration format. This time-based policy ensures that data is automatically deleted after a specified period, freeing up valuable storage space for new data.

```
views:
  - name: “firstView”
    retentionPolicies:
      - name: “timebased”
        config:
          duration: “PT5M”
```
duration:  "PT5M"

As an example, the time-based retention configuration example above is set up to ensure that data is automatically deleted after 5 minutes (PT5M).

### Hosting the LDES stream SHACL shape

LDES server facilitates hosting an LDES stream SHACL shape. SHACL Shapes Constraint Language is a language used to validate RDF graphs against a set of conditions provided as shapes and other constructs in an RDF graph. SHACL shapes can be used to describe data graphs that satisfy these conditions, beyond validation, such as for user interface building, code generation, and data integration.

### Hosting DCAT metadata

DCAT is a standardised RDF vocabulary for data catalogues on the Web, allowing easy interoperability between catalogues. Using a standard schema, DCAT enhances discoverability and facilitates federated search across multiple catalogues.

LDES server facilitates hosting DCAT metadata when publishing an LDES. The DCAT metadata can be represented as linked data events and published on an LDES server. The LDES server can then serve as a real-time data source for DCAT metadata.

<https://github.com/Informatievlaanderen/VSDS-LDESServer4J#example-serving-dcat-metadata>

**Example Serving DCAT Metadata**

Supported file formats: .ttl, .rdf, .nq and .jsonld Templates for configuring the DCAT metadata can be found [here](https://github.com/Informatievlaanderen/VSDS-LDESServer4J/blob/main/templates/dcat).


```
ldes:
 dcat: { path of file or url containing DCAT metadata, e.g. “dcat/metadata.ttl”}
```

## (Re)publish an LDES stream

An LDES server facilitate the possibility to publish and republish an LDES stream.

![](/images/LDES%20server.png)


# Authentication and authorisation


Authentication and authorisation enable secure data publication with restricted access.

In an OAuth2 gateway scenario, the gateway acts as an intermediary, handling authentication and authorisation, issuing access tokens, and granting access based on permissions.

![](/images/authorisation.png)

The user is prompted to enter their login credentials, which are authenticated by the [OAuth2](https://oauth.net/2/) gateway. If authentication is successful, the user can access the LDES stream.

For governmental organisations [Access (ACM) and User Management (IDM)](https://www.vlaanderen.be/acm-idm-standaard-aansluitingsproces) can be used.