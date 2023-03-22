---
sort: 8
---
# PUBLISH LDES


## LDES server

The Linked Data Event Stream (LDES) [server](https://github.com/Informatievlaanderen/VSDS-LDESServer4J) is a configurable component that can be used to ingest, store, and (re-)publish an LDES. The LDES server was built in the context of the VSDS project to exchange Open Data easily.


<p align="center"><img src="/images/LDES%20server.png" width="60%" text-align="center"></p>

The server can tailor its functionality to meet the organisation's specific needs. Functionalities include **retention policy**, **fragmentation** and **pagination**  for managing and processing large amounts of data more efficiently and ensuring the efficient use of storage. 

![](file:///C:/Users/samue/AppData/Local/Temp/msohtmlclip1/01/clip_image004.jpg)



### Ingesting sources (HTTP in)

The LDES server is able to receive data via HTTP ingestion. Specifically, the server expects an LDES-compliant member to be sent as input via a POST request. This member must be a version object of an entity that adheres to the LDES standard.

However, if the data stream is not yet LDES compliant, such as a state object data stream, it must first be converted to a version object. This essential step ensures data compatibility and adherence to the LDES standards.

Once the data stream is converted to a version object and the LDES-compliant member is received by the LDES server, the server can effortlessly publish the LDES member as part of the LDES.

When starting up the LDES server, the server checks if a SHACL shape (Shapes Constraint Language) is provided before proceeding with the ingestion process. A SHACL shape is used to validate members against predefined validation rules and to verify the conformity of RDF data with these rules.

The SHACL shape specifies the properties of an RDF resource and the rules that must be followed to ensure the data adheres to the expected structure and semantics. It defines properties such as required properties, allowed property values, and the data types expected for the properties.

For more information about the SHACL shape and its structure, **link:** **go to ****2.3**** SHACL Shape.**

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

### Fragmentation

To reduce the amount of data consumers need to replicate or to speed up certain queries, the LDES server can be configured with several fragmentations. Fragmentations are similar to indexes in databases but then published on the Web. The RDF predicate on which the fragmentation must be applied is defined through configuration.

<p align="center"><img src="/images/fragmentation.png" width="60%" text-align="center"></p>

The fragmenting of a Linked Data Event Stream (LDES) is a crucial technique for managing and processing large amounts of data more efficiently. There are three main methods of fragmentation: **geospatial**,** time-based**, and **substring** fragmentation.


**Partitioning**

When applying partitioning, the LDES server will create fragments based on the order of arrival of the LDES member. This fragmentation is considered to be the most basic and default fragmentation.\
The members that arrive first on the LDES server are added to the first page, while the latest members are always included on the latest page.



```yaml
name: “pagination”
config:
  memberLimit: { Mandatory: member limit > 0 }
```

[**Substring fragmentation**](https://github.com/Informatievlaanderen/VSDS-LDESServer4J/tree/main/ldes-fragmentisers/ldes-fragmentisers-substring)

Substring fragmentation involves dividing the data stream into smaller pieces based on specific substrings, or patterns, within the data. 

Example of substring fragmentation configuration file

```yaml
name: “substring”
config:
  fragmenterProperty: { Defines which property will be used for bucketizing }
  memberLimit: { member limit > 0 }
```
[**Time-based fragmentation**](https://github.com/Informatievlaanderen/VSDS-LDESServer4J/tree/main/ldes-fragmentisers/ldes-fragmentisers-timebased)

Time-based fragmentation has not yet been implemented.

<p align="center"><img src="/images/temporal.png" width="60%" text-align="center"></p>

Example of a time-based fragmentation configuration file



```yaml
name: “timebased”
config:
  memberLimit: { member limit > 0 }
```

[**Geospatial fragmentation**](https://github.com/Informatievlaanderen/VSDS-LDESServer4J/tree/main/ldes-fragmentisers/ldes-fragmentisers-geospatial)

Geospatial fragmentation involves dividing the data stream into smaller pieces based on geographical information, allowing organisations to process and analyse data within specific geographic areas in real-time. 

<p align="center"><img src="/images/geospatial.png" width="60%" text-align="center"></p>

The geospatial fragmentation follows the "Slippy Maps" principle. A zoom level is set through configuration, and the "world" is divided into tiles based on this zoom level. The number of tiles is 2^2n^ (where n = zoom level). An RDF predicate must also be configured for this fragmentation, determining on which property of the LDES member the fragmentation should be applied ....

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

### Retention policy


A [retention policy](https://github.com/Informatievlaanderen/VSDS-LDESServer4J#example-retention) determines how long data will be kept and stored. Its purpose is to ensure the efficient use of storage resources by controlling data growth over time. Setting a retention policy per view to minimise storage fill-up is possible.

<p align="center"><img src="/images/retention_policy.png" width="60%" text-align="center"></p>

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

### Hosting the LDES stream SHACL shape

LDES server facilitates hosting an LDES stream SHACL shape. SHACL Shapes Constraint Language is a language used to validate RDF graphs against a set of conditions provided as shapes and other constructs in an RDF graph. SHACL shapes can be used to describe data graphs that satisfy these conditions, beyond validation, such as for user interface building, code generation, and data integration.

### Hosting DCAT metadata

DCAT is a standardised RDF vocabulary for data catalogues on the Web, allowing easy interoperability between catalogues. Using a standard schema, DCAT enhances discoverability and facilitates federated search across multiple catalogues.

LDES server facilitates hosting DCAT metadata when publishing an LDES. The DCAT metadata can be represented as linked data events and published on an LDES server. The LDES server can then serve as a real-time data source for DCAT metadata.


More information about this can be found [here](https://github.com/Informatievlaanderen/VSDS-LDESServer4J#example-serving-dcat-metadata)

**Example Serving DCAT Metadata**

Supported file formats: .ttl, .rdf, .nq and .jsonld Templates for configuring the DCAT metadata can be found [here](https://github.com/Informatievlaanderen/VSDS-LDESServer4J/blob/main/templates/dcat).


```yaml
ldes:
 dcat: { path of file or url containing DCAT metadata, e.g. “dcat/metadata.ttl”}
```

### (Re)publish an LDES stream

An LDES server facilitate the possibility to publish and republish an LDES stream.

<p align="center"><img src="/images/LDES%20server.png" width="60%" text-align="center"></p>


# Authentication and authorisation


Authentication and authorisation enable secure data publication with restricted access.

In an OAuth2 gateway scenario, the gateway acts as an intermediary, handling authentication and authorisation, issuing access tokens, and granting access based on permissions.

<p align="center"><img src="/images/authorisation.png" width="60%" text-align="center"></p>

The user is prompted to enter their login credentials, which are authenticated by the [OAuth2](https://oauth.net/2/) gateway. If authentication is successful, the user can access the LDES stream.

For governmental organisations [Access (ACM) and User Management (IDM)](https://www.vlaanderen.be/acm-idm-standaard-aansluitingsproces) can be used.
<br>

---

<br>


## Set-up of the LDES Server

The current implementation consists of 6 modules:

- `ldes-server-application` which starts the spring boot application
- `ldes-server-domain` which contains the domain logic of the ldes server
- `ldes-server-infra-mongo` which allows to store ldes members and fragments in a mongoDB
- `ldes-server-port-ingestion-rest` which allows to ingest ldes members via HTTP
- `ldes-server-port-fetch-rest` which allows to retrieve fragments via HTTP
- `ldes-fragmentisers` which support different types of fragmentations

The modules `ldes-server-infra-mongo`, `ldes-server-port-ingestion-rest` and `ldes-server-port-fetch-rest` are built so
that they can be replaced by other implementations without the need for code changes in ldes-server-domain.

We'll show you how to set up your own LDES server both locally and via Docker using a mongo db for storage and a rest
endpoint for ingestion and fetch.
Afterwards, you can change storage, ingestion and fetch options by plugging in other components.

### Locally

**Maven**

To locally run the LDES server in Maven, move to the `ldes-server-application` directory and run the Spring Boot
application.

```note
This can be done as follows: \
**_NOTE:_**  Due to an authorisation issue in GitHub packages, the easiest way is to first build the project using the
following command: \
`mvn install`

```

```shell
cd ldes-server-application
```

```mvn
mvn spring-boot:run -P{profiles (comma separated with no spaces) }
```

for example:
```mvn
mvn spring-boot:run -P{fragmentation-pagination,http-fetch,http-ingest,queue-none,storage-mongo}
```

<br>
<br>




To enrich the server, certain Maven profiles can be activated:

**Profiles**

| Profile Group                           | Profile Name               | Description                                                     | Parameters                                                                  | Further Info                                                                                                                        |
|-----------------------------------------|----------------------------|-----------------------------------------------------------------|-----------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| **HTTP Endpoints (Fetch/Ingestion)**    | http-ingest                | Enables a HTTP endpoint for to insert LDES members.             | [HTTP configuration](#example-http-ingest-fetch-configuration)              | Endpoint:<br><br>- URL: /{ldes.collection-name}<br>- Request type: POST<br>- Accept: "application/n-quads", "application/n-triples" |
| **HTTP Endpoints (Fetch/Ingestion)**    | http-fetch                 | Enables a HTTP endpoint to retrieve LDES fragments              | [Example Views Configuration](#example-views-configuration)                 | Endpoint:<br>- URL: /{views.name}<br><br>- Request type: GET<br>- Accept: "application/n-quads", "application/ld+json"              |
| **Storage**                             | storage-mongo              | Allows the LDES server to read and write from a mongo database. | [Mongo configuration](#example-mongo-configuration)                         |                                                                                                                                     |
| **Timebased Fragmentation[DEPRECATED]** | fragmentation-timebased    | Supports timebased fragmentation.                               | [Timebased fragmentation configuration](#example-timebased-fragmentation)   |                                                                                                                                     |
| **Geospatial Fragmentation**            | fragmentation-geospatial   | Supports geospatial fragmentation.                              | [Geospatial fragmentation configuration](#example-geospatial-fragmentation) |                                                                                                                                     |
| **Substring Fragmentation**             | fragmentation-substring    | Supports substring fragmentation.                               | [Substring fragmentation configuration](#example-substring-fragmentation)   |                                                                                                                                     |
| **Pagination Fragmentation**            | fragmentation-pagination   | Supports pagination.                                            | [Pagination configuration](#example-pagination)                             | The pagenumbers start with pagenumber 1                                                                                             |
| **Ldes-queues**                         | queue-none                 | Members are fragmented immediately.                             | N/A activating the profile is enough                                        |                                                                                                                                     |
| **Ldes-queues**                         | queue-in-memory            | Members are queued in memory before fragmentation.              | N/A activating the profile is enough                                        |                                                                                                                                     |

The main functionalities of the server are ingesting and fetching, these profiles depend on other supporting profiles to function properly:
- http-ingest: requires at least one queue, one fragmentation and one storage profile.
- http-fetch: requires at least one storage profile

**Application Configuration**

Below are properties that are needed when applying certain profiles.
These need to be added in the `application.yml` file in `ldes-server-application/src/main/resources`. (If the file does not exist, create it)

A minimal working example of the application.yml can be found [here](ldes-server-application/examples/minimal-config-application.yml). This works for both fetching and ingesting. 

The server allows configurable fragment refresh times with the max-age and max-age-immutable options. These values will be sent with the Cache-Control header in HTTP responses.


<ins>Example HTTP Ingest-Fetch Configuration:</ins>

  ```yaml
  server.port: { http-port }
  ldes:
    collection-name: { short name of the collection, cannot contain characters that are used for url interpretation, e.g. '$', '=' or '&' }
    host-name: { hostname of LDES Server }
    member-type: { Defines which syntax type is used to define the member id e.g. "https://data.vlaanderen.be/ns/mobiliteit#Mobiliteitshinder" }
    timestamp-path: { SHACL property path to the timestamp when the version object entered the event stream. }
    version-of: { SHACL property path to the non-versioned identifier of the entity. }
    validation:
      shape: { URI to defined shape }
      enabled: { Enables/Disables shacl validation on ingested members }
  rest:
    max-age: { time in seconds that a mutable fragment can be considered up-to-date, default when omitted: 60 }
    max-age-immutable: { time in seconds that an immutable fragment should not be refreshed, default when omitted: 604800 }
  ```

<ins>Example Mongo Configuration:</ins>

  ```yaml
  spring.data.mongodb:
    uri: mongodb://{docker-hostname}:{port}
    database: { database name }
  ```


<ins>Example Views Configuration:</ins>

  ```yaml
    views:
      - name: {name of the view}
        fragmentations:
          - name: {type of fragmentation}
            config:
              {Map of fragmentation properties}
  ```

<ins>An example of a view configuration with two view is shown below:</ins>

  ```yaml
  views:
    - name: "firstView"
      fragmentations:
        - name: "geospatial"
          config:
            maxZoomLevel: 15
            fragmenterProperty: "http://www.opengis.net/ont/geosparql#asWKT"
        - name: "pagination"
          config:
            memberLimit: 5
    - name: "secondView"
      fragmentations:
        - name: "pagination"
          config:
            memberLimit: 3
  ```
<ins>Example Retention:</ins>

To reduce storage fill up, it is possible to set a retention policy per view.
As of now, there is only a timebased retention possible which can be configured with a [ISO 8601 duration](https://tc39.es/proposal-temporal/docs/duration.html)

  ```yaml
  views:
    - name: "firstView"
      retentionPolicies:
        - name: "timebased"
          config:
            duration: "PT5M"
  ```

<ins>Example Timebased Fragmentation:</ins>

This fragmentation is DEPRECATED, more information can be found [here](ldes-fragmentisers/ldes-fragmentisers-timebased/README.MD)

  ```yaml
  name: "timebased"
  config:
    memberLimit: { member limit > 0 }
  ```

<ins>Example Geospatial Fragmentation:</ins>

Full documentation for geospatial fragmentation can be found [here](ldes-fragmentisers/ldes-fragmentisers-geospatial/README.MD)

  ```yaml
  name: "geospatial"
  config:
    maxZoomLevel: { Required zoom level }
    fragmenterProperty: { Defines which property will be used for bucketizing }
  ```

<ins>Example Substring Fragmentation:</ins>

Full documentation for substring fragmentation can be found [here](ldes-fragmentisers/ldes-fragmentisers-substring/README.MD)

  ```yaml
  name: "substring"
  config:
    fragmenterProperty: { Defines which property will be used for bucketizing }
    memberLimit: { member limit > 0 }
  ```

<ins>Example Pagination:</ins>

Full documentation for pagination fragmentation can be found [here](ldes-fragmentisers/ldes-fragmentisers-pagination/README.MD)

  ```yaml
  name: "pagination"
  config:
    memberLimit: { member limit > 0 }
  ```

<ins>Example Serving Static Content:</ins>

  ```yaml
spring:
  mvc:
    static-path-pattern: { pattern used in url for static content, e.g. /content/** for serving on http://localhost:8080/content/ }
  web:
    resources:
      static-locations: { source folder of static content, e.g. file:/opt/files }
  ```

<ins>Example Serving DCAT Metadata:</ins>

Supported file formats: .ttl, .rdf, .nq and .jsonld
Templates for configuring the DCAT metadata can be found [here](templates/dcat)

  ```yaml
ldes:
   dcat: { path of file or url containing DCAT metadata, e.g. "dcat/metadata.ttl"  }
  ```
<br>

<br>

---

### Docker Setup

**Docker-compose**

There are 2 files where you can configure the dockerized application:

- [The config files](#the-config-files)
- [The docker compose file](#the-docker-compose-file)

**The Config Files**

Runtime settings can be defined in the configuration files. Use [config.env](docker-compose/config.env) for a public
setup and be sure not to commit this file, as it contains secrets. For a local setup,
use [config.local.env](docker-compose/config.local.env).

**The docker compose File**

Change the `env_file` to `config.env` or `config.local.env` in [compose.yml](compose.yml) according to
your needs.

**Starting the Dockerized Application**

Run the following commands to start the containers:

```bash
COMPOSE_DOCKER_CLI_BUILD=1 docker-compose build
docker-compose up
```
```note
**Note**: Using Docker Desktop might fail because of incorrect environment variable interpretation.
 
`unexpected character "-" in variable name near ...`
```
<br>

<br>

---

## Developer Information

**How To Build**

For compilation of the source code, execute the following command

```
mvn clean compile
```

**How To Test and View Coverage**

Below the three options to run tests (only unit test, only integration tests and both unit and integration tests) are
listed.
To view the coverage of each option it's sufficient to add the option `-Pcoverage` after the command.
This generates a file in the `target` folder called `jacoco.exec`. This file reports the code coverage.
The combined coverage can also be seen via SonarCloud.

<ins>Unit and Integration Tests</ins>

For running all the tests of the project, execute the following command

```
mvn clean verify
```

<ins>Only Unit Tests</ins>

For running the unit tests of the project, execute the following command

```
mvn clean verify -Dintegrationtestskip=true
```

<ins>Only Integration Tests</ins>

For running the integration tests of the project, execute the following command

```
mvn clean verify -Dunittestskip=true
```

<ins>Auto-Configurable Modules</ins>

- ldes-server-infra-mongo
- ldes-server-port-ingestion-rest
- ldes-server-port-publication-rest
- ldes-fragmentisers-timebased
- ldes-fragmentisers-geospatial
- ldes-fragmentisers-pagination
- ldes-fragmentisers-substring

**Tracing and Metrics**

Additionally, it is possible to keep track of metrics and tracings of the LDES Server.
This will be done through a Zipkin exporter for traces and a Prometheus endpoint for Metrics.

The exposed metrics can be found at `/actuator/metrics`.

Both traces and metrics are based on [OpenTelemetry standard](https://opentelemetry.io/docs/concepts/what-is-opentelemetry/)


To achieve this, the following properties are expected

<ins>Local Tracing and Metrics</ins>

```yaml
management:
  tracing:
    sampling:
      probability: 1.0
  zipkin:
    tracing:
      endpoint: "zipkin endpoint of collector"
  endpoints:
    web:
      exposure:
        include: 
          - prometheus
```

The export of traces can be disabled with the following parameter:

```yaml
management:
  tracing:
    enabled: false
  ```

<ins>Using Docker</ins>

```
SPRING_SLEUTH_OTEL_EXPORTER_JAEGER_ENDPOINT="endpoint of collector"
MANAGEMENT_ENDPOINTS_WEB_EXPOSURE_INCLUDE="prometheus"
MANAGEMENT_TRACING_SAMPLING_PROBABILITY="1.0"
MANAGEMENT_ZIPKIN_TRACING_ENDPOINT="zipkin endpoint of collector"
```
The export of traces can be disabled with the following parameter:

```
MANAGEMENT_TRACING_ENABLED=false
```

**Health and Info**

To allow more visibility for the application, it is possible to enable a health and info endpoint.

This health endpoint provides a basic JSON output that can be found at `/actuator/health` that provides a summary of the status of all needed services.

An additional info endpoint is also available which shows which version of the application running and its deployment date.

<ins>Local Health and Info</ins>

The following config allows you to enable both the info and health endpoints.
```yaml
management:
  endpoints:
    web:
      exposure:
        include:
          - health
          - info
  health:
    defaults:
      enabled: false
    mongo:
      enabled: true
  endpoint:
    health:
      show-details: always
  ```

<ins>Docker</ins>

```
MANAGEMENT_ENDPOINTS_WEB_EXPOSURE_INCLUDE="health, info"
MANAGEMENT_HEALTH_DEFAULTS_ENABLED=false
MANAGEMENT_HEALTH_MONGO_ENABLED=true
MANAGEMENT_ENDPOINT_HEALTH_SHOW-DETAILS="always"
```

**Logging**


The logging of this server is split over the different logging levels according to the following guidelines.

- TRACE: NONE
- DEBUG: Standard operations like: create fragment, ingest member, assign member to fragment
- INFO: NONE
- WARN: Potentially unintended operations like: Duplicate Member Ingest, ...
- ERROR: All Exceptions

<ins>Logging configuration</ins>

The following config allows you to output logging to the console. Further customization of the logging settings can be done using the logback properties.
```yaml
logging:
  pattern:
    console: "%d %-5level %logger : %msg%n"
  level:
    root: INFO
```

The following config enables and exposes the loggers endpoint.
```yaml
management:
  endpoint:
    loggers:
      enabled: true
  endpoints:
    web:
      exposure:
        include:
          - loggers
```

To change the logging level of the application at runtime, you can send the following POST request to the loggers endpoint.
Replace [LOGGING LEVEL] with the desired logging level from among: TRACE, DEBUG, INFO, WARN, ERROR.
```
curl -i -X POST -H 'Content-Type: application/json' -d '{"configuredLevel": "[LOGGING LEVEL]"}'
  http://localhost:8080/actuator/loggers/ROOT
```