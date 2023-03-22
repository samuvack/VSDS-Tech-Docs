---
sort: 2
---

# INTRODUCTION


<cite>"Linked Data Event Stream as the base API for publishing open data."</cite>

What is the best API to publish an open dataset?

Data publishers often struggle to meet user needs and expectations by creating and maintaining multiple querying APIs for their datasets. These APIs may include geospatial queries, graph queries, keyword searches, and others. However, there are some drawbacks to this approach. For example, keeping multiple APIs online can be costly regarding resources and effort. Data publishers must ensure their APIs are always up-to-date with the latest standards and technologies.

As new trends emerge, old APIs may become obsolete and incompatible, creating legacy issues for data consumers who may have to switch to new APIs or deal with outdated data formats and functionalities. Moreover, the existing APIs limit data reuse and innovation since the capabilities and limitations of these APIs constrain data consumers. They cannot create their views or indexes on top of the data using any technology they prefer.

To overcome these challenges, Linked Data Event Streams (LDES) provides a generic and flexible base API for open datasets. With LDES, data consumers can replicate the history of a dataset and stay in sync with it as well.

<cite>"Using LDES as the base API for open datasets, data publishers can avoid the challenges associated with maintaining multiple querying APIs and foster data reuse and innovation."</cite>

Data consumers often have to deal with multiple versions or copies of a dataset that are not consistent or synchronised. These versions or copies may have different snapshots or deltas published at other times or frequencies. However, there are some drawbacks to this approach as well. For example, data consumers must track when and how each version or copy was created and updated. They also have to compare different versions or copies to identify changes or conflicts in the data. Data consumers may encounter errors or inconsistencies due to outdated or incomplete versions or copies. They may also miss significant changes or updates in data not reflected in their versions or copies.

LDES can also help overcome these challenges by providing a generic and flexible base API for open datasets. With LDES, data consumers can use RDF tools and standards to follow the history and provenance of each object in an LDES. They can also use RDF tools and standards to compare different things in an LDES based on their properties and relations. Data consumers only download and store changes in data rather than full snapshots or deltas, reducing bandwidth and storage requirements. They only process and filter relevant data based on their views and indexes on top of the main event stream. LDES also ensures that data consumers consistently access the latest and most complete version of data and receive notifications about significant changes or updates in data.

<cite>"Anyone can build their querying interface and keep it up to date"</cite>

<p align="center"><img src="/VSDS-Tech-Docs/images/LDES_API.png"  width="100%" text-align="center"></p>


Linked Data Event Streams (LDES) try to fix this situation by providing a generic and flexible base API for open datasets.

Linked Data Event Streams (LDES) fix the 'Maintenance hell' by allowing data consumers to replicate and synchronise with a reference dataset by following all relations starting from a view. LDES have some advantages over other querying APIs:

- LDES only publishes changes in data rather than full snapshots or deltas, reducing bandwidth and storage requirements.

- LDES allows data consumers to create views and indexes on top of the main event stream using any preferred technology.

Using LDES as the base API for open datasets, data publishers can avoid maintenance hell and foster data reuse and innovation.

Linked Data Event Streams (LDES) fix the 'Replication hell' by providing a generic and flexible base API for open datasets. LDES allow data consumers to replicate and synchronise with a reference dataset by following all relations starting from a view. 

```note
LDES have some advantages over other versions or copies:

- Data consumers can use RDF tools and standards to follow the history and provenance of each object in an LDES. They can also use RDF tools and standards to compare different things in an LDES based on their properties and relations.

- Data consumers only download and store changes in data rather than full snapshots or deltas, reducing bandwidth and storage requirements. They only process and filter relevant data based on their views and indexes on top of the main event stream.

- Data consumers consistently access the latest and most complete version of data through an LDES. They receive notifications about significant changes or updates in data through an LDES.
```