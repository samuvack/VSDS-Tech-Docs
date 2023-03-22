---
sort: 9
---

# CONSUME LDES



## LDES Client



![](/VSDS-Tech-Docs/images/LDES%20client.png)

The [LDES CLIENT](https://github.com/Informatievlaanderen/VSDS-Linked-Data-Interactions) is designed for both replication and synchronisation, meaning that the client can retrieve members of an LDES but also checks regularly if new members are added, and fetches them, allowing data consumers to stay up to date with the dataset.

To understand how an LDES client functions, it is important to understand how LDESes are published on the Web. To publish an LDES, the principle of [Linked Data Fragments](https://linkeddatafragments.org/specification/linked-data-fragments/) is used, meaning that data is put in one or more multiple fragments and semantic meaningful links are created between these fragments. This allows clients to follow these links and discover more data. However, the most important aspect for the LDES client is the concept of mutable and immutable fragments. When publishing an LDES stream, it is a common configuration to have a maximum number of members per fragment. Once a fragment reaches the page limit, it can be considered *immutable,* which is made clear by adding the cache header **'Cache-control: immutable'** to the fragment.

This information is important for the LDES client, because it only must fetch an immutable fragment once, while mutable (thus, still changing) fragments must be regularly polled to check for new members.

### Replication


In order to start the replication of an LDES, data consumers must configure the LDES client with an LDES *view* endpoint. If the data consumer configures the LDES endpoint, which possibly describes multiple views, the LDES client will use the first view it receives to start the replication.

![](/VSDS-Tech-Docs/images/)

Whenever a client visits a fragment, it parses the contents to RDF and looks for triples with the **tree:member** predicate to discover members of the LDES. In addition, the client also searches for triples with a **tree:relation** predicate, indicating relations to other fragments and adds it to its queue of to-be-fetched fragments, if the fragment was not already fetched earlier. For every fragment, the client checks the *response headers*, looking for a possible '*Cache-control: immutable*', indicating that the fragment does not need to be polled again.

### Synchronisation


In addition to replication, the LDES client keeps track of all *mutable* fragments and periodically polls them, and checks if new members were added. To further optimise the synchronisation process, the LDES client reads the *'Cache-control: max-age'* value from the response headers, which specifies the time period for which the LDES fragment remains valid. This allows the LDES client to schedule periodic polling more efficiently.

By utilising the response headers provided by the LDES server, the LDES client can effectively manage the synchronisation of the LDES stream, while minimising the amount of processing required. This makes the LDES client an efficient and reliable tool for processing and managing Linked Data Event Streams.



### Persisting the state


In addition to the functionalities above, the LDES client maintains an SQLite database of immutable and mutable fragment IDs. For mutable fragments, member IDs are also stored in the database, ensuring that a member is not processed twice.

![](/VSDS-Tech-Docs/images/State.png)

This way, the LDES client also functions as a gatekeeper, allowing it to continue where it stopped without having to read the LDES from the beginning.




### Quickstart



