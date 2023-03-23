---
sort: 9
---

# LDES CLIENT


<p align="center"><img src="/VSDS-Tech-Docs/images/LDES%20client.png" width="60%" text-align="center"></p>

The [LDES CLIENT](https://github.com/Informatievlaanderen/VSDS-Linked-Data-Interactions) is designed for both replication and synchronisation, meaning that the client can retrieve members of an LDES but also checks regularly if new members are added, and fetches them, allowing data consumers to stay up to date with the dataset.

To understand how an LDES client functions, it is important to understand how LDESes are published on the Web. To publish an LDES, the principle of [Linked Data Fragments](https://linkeddatafragments.org/specification/linked-data-fragments/) is used, meaning that data is put in one or more multiple fragments and semantic meaningful links are created between these fragments. This allows clients to follow these links and discover more data. However, the most important aspect for the LDES client is the concept of mutable and immutable fragments. When publishing an LDES stream, it is a common configuration to have a maximum number of members per fragment. Once a fragment reaches the page limit, it can be considered *immutable,* which is made clear by adding the cache header **'Cache-control: immutable'** to the fragment.

This information is important for the LDES client, because it only must fetch an immutable fragment once, while mutable (thus, still changing) fragments must be regularly polled to check for new members.

## Replication


In order to start the replication of an LDES, data consumers must configure the LDES client with an LDES *view* endpoint. If the data consumer configures the LDES endpoint, which possibly describes multiple views, the LDES client will use the first view it receives to start the replication.

![](/VSDS-Tech-Docs/images/)

Whenever a client visits a fragment, it parses the contents to RDF and looks for triples with the **tree:member** predicate to discover members of the LDES. In addition, the client also searches for triples with a **tree:relation** predicate, indicating relations to other fragments and adds it to its queue of to-be-fetched fragments, if the fragment was not already fetched earlier. For every fragment, the client checks the *response headers*, looking for a possible '*Cache-control: immutable*', indicating that the fragment does not need to be polled again.

## Synchronisation


In addition to replication, the LDES client keeps track of all *mutable* fragments and periodically polls them, and checks if new members were added. To further optimise the synchronisation process, the LDES client reads the *'Cache-control: max-age'* value from the response headers, which specifies the time period for which the LDES fragment remains valid. This allows the LDES client to schedule periodic polling more efficiently.

By utilising the response headers provided by the LDES server, the LDES client can effectively manage the synchronisation of the LDES stream, while minimising the amount of processing required. This makes the LDES client an efficient and reliable tool for processing and managing Linked Data Event Streams.



## Persisting the state


In addition to the functionalities above, the LDES client maintains an SQLite database of immutable and mutable fragment IDs. For mutable fragments, member IDs are also stored in the database, ensuring that a member is not processed twice.

<p align="center"><img src="/VSDS-Tech-Docs/images/State.png" width="60%" text-align="center"></p>

This way, the LDES client also functions as a gatekeeper, allowing it to continue where it stopped without having to read the LDES from the beginning.



## Linked Data Interactions

[This module](https://github.com/Informatievlaanderen/VSDS-Linked-Data-Interactions/tree/main/ldi-core/ldes-client) contains the LDES client SDK that replicates and synchronises an LDES and keeps a (non-persisted) state for that process. Wrappers can call the SDK to do the work of scheduling fragment fetching and extracting members.

The main goal for the LDES Client SDK is to replicate an LDES and then synchronise it.

This is achieved by configuring the processor with an initial fragment URL. When the processor is triggered, the fragment will be processed, and all relations will be added to the (non-persisted) queue.

As long as the processor runs, a queue that accepts new fragments to process is maintained. The processor also keeps track of the mutable and immutable fragments already processed.

It will be ignored when an attempt is made to queue a known immutable fragment. Fragments in the mutable fragment store will be queued when they're expired. Should a fragment be processed from a stream that does not set the max-age in the Cache-control header, a default expiration interval will be used to set an expiration date on the fragment.

Processed members of mutable fragments are also kept in state. They are ignored if presented more than once.



## E2E Client Sink


The E2E Client Sink is a small HTTP server used for E2E testing the LDES client NiFi processor.

### Docker


The sink can be run as a [Docker container](https://github.com/Informatievlaanderen/VSDS-LDES-E2E-testing/tree/main/ldes-client-sink#docker), using a pre-built container or after creating a Docker image for it locally. The Docker container will keep running until stopped.

To create a Docker image, run the following command:

```source-shell
docker build --tag vsds/ldes-client-sink .
```

You can use a MongoDB as a member store. Please ensure you run a MongoDB instance locally or use an online instance. Configure the Docker container to use that instance or configure and use the [Docker compose file](https://github.com/Informatievlaanderen/VSDS-LDES-E2E-testing/blob/main/ldes-client-sink/docker-compose.yml) that has been provided. Alternatively you can use an in-memory database (simple object store).

To run the sink Docker image mapped on port 9000 using a MongoDB, you can use:

```source-shell
docker run -d -p 9000:80 --add-host=host.docker.internal:host-gateway\
-e MEMBER_TYPE="http://schema.org/Person" -e COLLECTION_NAME="cartoons"\
-e CONNECTION_URI="mongodb://host.docker.internal:27017" -e DATABASE_NAME="test"\
vsds/ldes-client-sink
```

Alternatively, with an in-memory database:

```source-shell
docker run -d -p 9000:80 --add-host=host.docker.internal:host-gateway\
-e MEMBER_TYPE="http://schema.org/Person" -e COLLECTION_NAME="cartoons"\
-e MEMORY=true\
vsds/ldes-client-sink
```

The Docker run command will return a container ID (e.g. `0cc5d65d8108f8e91778a0a4cdb6504a2b3926055ce10cb899dceb98db4c3eef`), which you need to stop the container.

Alternatively you can run `docker ps` to retrieve the (short version of the) container ID.

```
CONTAINER ID   IMAGE                   COMMAND                  CREATED          STATUS          PORTS                  NAMES
0cc5d65d8108   vsds/ldes-client-sink   "/usr/bin/dumb-init ..."   12 seconds ago   Up 11 seconds   0.0.0.0:9000->80/tcp   intelligent_bell

```

To stop the container, you need to call the stop command with the (long or short) container ID, e.g. `docker stop 0cc5d65d8108`

**Docker compose**

For your convenience a [Docker compose file](https://github.com/Informatievlaanderen/VSDS-LDES-E2E-testing/blob/main/ldes-client-sink/docker-compose.yml) is provided containing the client sink and a MongoDB store, and a [.env](https://github.com/Informatievlaanderen/VSDS-LDES-E2E-testing/blob/main/ldes-client-sink/.env) file with environment variables used for building and running the containers. The sink variables typically need tuning for your use case. The easiest way to provide these is to copy the `.env` file into a file named `user.env` and change the variables as required. Then you can run the following command to build and run the [Docker containers](https://github.com/Informatievlaanderen/VSDS-LDES-E2E-testing/tree/main/ldes-client-sink#docker-compose):

```source-shell
docker compose --env-file user.env up
```

### Build


The sink is implemented as a [Node.js](https://nodejs.org/en/) application. You need to run the following commands to [build it](https://github.com/Informatievlaanderen/VSDS-LDES-E2E-testing/tree/main/ldes-client-sink#build):

```source-shell
npm i
npm run build
```

**Run**


The sink uses a MongoDB as permanent storage to allow for large data sets, so before running the sink, make sure your MongoDB instance is running.

The sink takes the following command line arguments:

-   `--member-type` defines the member type to use to determine the member's ID (subject value), no default
-   `--port=<port-number>` allows to set the port, defaults to `9000`
-   `--host=<host-name>` allows to set the hostname, defaults to `localhost`
-   `--silent=<true|false>` prevents any console debug output if true, defaults to false (not silent, logging all debug info)
-   `--connection-uri` allows to set the MongoDB connection URI, defaults to `mongodb://localhost:27017`
-   `--database-name` allows to set the MongoDB database name, defaults to `ldes_client_sink`
-   `--collection-name` allows to set the MongoDB collection name, defaults to `members`
-   `--memory` allows to use an in-memory database for smaller data sets instead of MongoDB, defaults to false

You can [run the sink](https://github.com/Informatievlaanderen/VSDS-LDES-E2E-testing/tree/main/ldes-client-sink#run) with one of the following command after building it:

```source-shell
node dist/server.js --member-type "http://schema.org/Person" --collection-name cartoons --memory true
```

This results in:

```
Arguments:  {
  _: [],
  'member-type': 'http://schema.org/Person',
  'collection-name': 'cartoons'
}
Sink listening at http://127.0.0.1:9000

```

**Usage**

The sink server accepts the following [REST calls](https://github.com/Informatievlaanderen/VSDS-LDES-E2E-testing/tree/main/ldes-client-sink#usage).

<ins>`GET /` -- Retrieve Number of Ingested Members</ins>

[Returns the number of members received, e.g.](https://github.com/Informatievlaanderen/VSDS-LDES-E2E-testing/tree/main/ldes-client-sink#get-----retrieve-number-of-ingested-members)

```source-shell
curl http://localhost:9000/
```

returns:

```source-json
{"cartoons":{"total":0}}
```

<ins>`POST /member` -- Ingest Members</ins>

[Ingests a member as quads](https://github.com/Informatievlaanderen/VSDS-LDES-E2E-testing/tree/main/ldes-client-sink#post-member----ingest-members) (mime-type: `application/n-quads`) or as triples (mime-type: `application/n-triples`) and returns the member ID (URI), e.g.

```source-shell
curl -X POST http://localhost:9000/member -H "Content-Type: application/n-quads" -d "@donald-duck.nq"
```

OR

```source-shell
curl -X POST http://localhost:9000/member -H "Content-Type: application/n-triples" -d "@donald-duck.nt"
```

returns:

```
http://example.org/id/cartoon-figure/donald-duck

```

<ins>`GET /member` -- Get Member List</ins>

[Returns the (limited) list of members (as local URLs), e.g.](https://github.com/Informatievlaanderen/VSDS-LDES-E2E-testing/tree/main/ldes-client-sink#get-member----get-member-list)

```source-shell
curl http://localhost:9000/member
```

returns (formatted for readability):

```source-json
{
  "cartoons": {
    "total": 1,
    "count": 1,
    "members": [
      "/member?id=http%3A%2F%2Fexample.org%2Fid%2Fcartoon-figure%2Fdonald-duck"
    ]
  }
}
```

<ins>`GET /member?id=<url-encoded-member-id>` -- Get Member Content</ins>

[Returns the member content as quads](https://github.com/Informatievlaanderen/VSDS-LDES-E2E-testing/tree/main/ldes-client-sink#get-memberidurl-encoded-member-id----get-member-content) (if ingested with mime-type: `application/n-quads`), e.g.

```source-shell
curl "http://localhost:9000/member?id=http%3A%2F%2Fexample.org%2Fid%2Fcartoon-figure%2Fdonald-duck"
```

returns (formatted for readability):

```
<http://example.org/id/cartoon-figure/donald-duck> <http://schema.org/name> "Donald Duck" <http://example.org/disney>.
<http://example.org/id/cartoon-figure/donald-duck> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://schema.org/Person> <http://example.org/disney>.

```

<ins>`DELETE /member` -- Remove all Members</ins>

[Removes all members, e.g.](https://github.com/Informatievlaanderen/VSDS-LDES-E2E-testing/tree/main/ldes-client-sink#delete-member----remove-all-members)

```source-shell
curl -X DELETE http://localhost:9000/member
```

Returns the amount of members deleted:

```source-json
{"count":1}
```