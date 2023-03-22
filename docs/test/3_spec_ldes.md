---
sort: 3
---

# SPECIFICATION

## What is a Linked data Event Stream


"LDES is a specification for publishing an append-only collection of immutable members. Consumers can then host one or more always in-sync Linked Data Fragments views on top of an Event Source."

A Linked Data Event Stream (LDES) is defined as a collection of immutable objects, referred to as *LDES members*.

These objects are described using a specific format called RDF triples, which stands for Resource Description Framework triples. RDF is one of the corner stones of Linked Data and on which Linked Data Event Streams continues to build.

```tip
More information on Linked Data can be found [here](https://www.w3.org/standards/semanticweb/data).
```

The LDES specification is based on a particular specification, called the [TREE specification](https://w3id.org/tree/specification). The TREE specification originates from the idea to provide an alternative to one-dimensional HTTP pagination. It allows to fragment a collection of items and interlink these fragments. Instead of linking to the next or previous page, the relation describes what elements can be found by following the link to another fragment. The LDES specification extends the TREE specification by stating that every item in the collection ***must***be immutable.

![](/VSDS-Tech-Docs/images/)



This specification is designed to be compatible with other specifications, such as Activity Streams Core, VOCAB-DCAT-2, LDP, and Shape Trees. This means you can use the LDES spec in combination with these other formats to create a more comprehensive and powerful data structure.

Linked Data Event Streams (LDES) apply --- as the term implies --- the Linked Data principles to data event streams. A data stream is typically a constant flow of distinct data points, each containing information about an event or change of state that originates from a system that continuously creates data. Some examples of data streams include sensor and other IoT data, financial data, etc.

```note
LDES has several technical advantages:

- LDES is a technical standard that allows data to be exchanged across silos using domain-specific ontology;

- LDES is one standard for both fast and slow-changing data;

- LDES offers a solution for managing historical records, versions, and retention policies efficiently.
```


A Linked Data Event Stream is a constant flow of immutable objects (such as version objects of addresses, sensor observations or archived representations) containing information about an event or change of state that originates from a system that continuously creates data.

Working with data, one of the main challenges we face is the burden of harvesting data without or with insufficient metadata. Acquiring data from a wide range of sources and formats (from data dumps in zip files to various databases, services, APIs, etc.) makes it a terrible task to transform and convert the data in a structured, matching structure. Gaining no accompanying information about the data's origin, context, or significance leads to misinterpreting data. In addition, working with outdated or incomplete information can lead to incorrect or misleading conclusions. As a result, there is a need for a specification where data is always up-to-date, metadata is included and can be easily linked with other data sets across the Web.

"Linked data event stream is the core API that solves all these needs."

LDES increases the usability and findability of data, as it comes in a uniform linked data standard published on an online URI endpoint. LDES holds historical data and ensures that the dataset is always up to date.

As a result, the user is always in sync with the publisher's dataset and can fetch all the historical data with its metadata.

In a nutshell, there are several reasons why there was a need to add Linked Data Event Streams as a specification:

- Linked Data is a powerful paradigm for representing and sharing data on the Web. Still, it has traditionally focused on representing static data rather than events or changes to that data.

- The use of event streams is becoming increasingly prevalent on the Web, as it enables applications to exchange information about changes to data in real-time efficiently.

- There was a need for a standard format for representing Linked Data events and changes so that different systems could easily interoperate and exchange this information.

- Linked Data Event Streams allow applications to subscribe to event streams and receive updates in real-time, which can be helpful in various applications, such as real-time data analysis, event-driven architecture, and more.

## Example of a Linked data Event Stream


State object

Version

CONTEXT

LDES

## HTML specification


```

@prefix example: <http://www.example.org/>.
@prefix ldes: <http://w3id.org/ldes#>.
@prefix tree: <https://w3id.org/tree#>.
@prefix dct: <http://purl.org/dc/terms/>.
@prefix xsd: <http://www.w3.org/2001/XMLSchema#>.

example:EventStream a ldes:Evenstream;
 tree:member example:member1, example:member2.

example:member1 dct:title “The first member”;
 dct:description “The first member of the Linked Data Event Stream”;
 dct:created "2023-01-02T10:00:00Z"^^xsd:dateTime.
 
example:member2 dct:title “The second member”;
 dct:description “The second member of the Linked Data Event Stream”;
 dct:created "2023-01-02T11:00:00Z"^^xsd:dateTime.
```

# Features

## Fragmentation and pagination

An LDES focuses on allowing clients to replicate a dataset's history and efficiently synchronise with its latest changes. Linked Data Event Streams may be fragmented when their size becomes too big for one HTTP response.



## Retention policy

A retention policy is a set of rules determining how long data should be kept or deleted. A retention policy can be applied to Linked Data Event Streams (LDES) to manage the storage and availability of data objects over time.



## SHACL

[SHACL (Shapes Constraint Language)](https://www.w3.org/TR/shacl/) is a standard for validating RDF data and ensuring that it conforms to a particular structure or shape. In the context of the Linked Data Event Stream (LDES), SHACL shapes are used to define the expected structure of the events in the stream, including their properties, relationships, and constraints.

By incorporating SHACL shapes, LDES provides a powerful tool for ensuring data quality and consistency, making it a reliable and trustworthy source of data for various applications. By defining a SHACL shape for the LDES, data producers can ensure that the events they generate adhere to the required structure, while data consumers can use the shape to validate and reason about the data they receive. LDES also supports dynamic SHACL shape discovery, allowing consumers to query the stream for relevant shapes and adapt to changes in the data over time.

## DCAT

[DCAT](https://www.w3.org/TR/vocab-dcat-3/) is an RDF vocabulary for data catalogues on the Web, enabling easy interoperability and discoverability of metadata for datasets, data services, and portals. It standardises properties for describing datasets, access information, and data services. By using DCAT, publishers can increase their datasets' exposure and facilitate data sharing and reuse. DCAT is extensible, maintained by the W3C, and the latest version is DCAT 3.