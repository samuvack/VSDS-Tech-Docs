---
sort: 4
---

Fragmentation and pagination
---------------------------------

An LDES focuses on allowing clients to replicate a dataset's history and efficiently synchronise with its latest changes. Linked Data Event Streams may be fragmented when their size becomes too big for one HTTP response.

**Link: Go to LDES server -> Fragmentation**

Retention policy
---------------------

A retention policy is a set of rules determining how long data should be kept or deleted. A retention policy can be applied to Linked Data Event Streams (LDES) to manage the storage and availability of data objects over time.

**Link: Go to LDES server ****à Retention policy**

SHACL
----------

[SHACL (Shapes Constraint Language)](https://www.w3.org/TR/shacl/) is a standard for validating RDF data and ensuring that it conforms to a particular structure or shape. In the context of the Linked Data Event Stream (LDES), SHACL shapes are used to define the expected structure of the events in the stream, including their properties, relationships, and constraints.

By incorporating SHACL shapes, LDES provides a powerful tool for ensuring data quality and consistency, making it a reliable and trustworthy source of data for various applications. By defining a SHACL shape for the LDES, data producers can ensure that the events they generate adhere to the required structure, while data consumers can use the shape to validate and reason about the data they receive. LDES also supports dynamic SHACL shape discovery, allowing consumers to query the stream for relevant shapes and adapt to changes in the data over time.

DCAT
---------

[DCAT](https://www.w3.org/TR/vocab-dcat-3/) is an RDF vocabulary for data catalogues on the Web, enabling easy interoperability and discoverability of metadata for datasets, data services, and portals. It standardises properties for describing datasets, access information, and data services. By using DCAT, publishers can increase their datasets' exposure and facilitate data sharing and reuse. DCAT is extensible, maintained by the W3C, and the latest version is DCAT 3.