---
sidebar_position: 3
title: "🧪 Usage Guide"
---

# 🧪 Usage Guide


import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

<div class="select-language">Select a language</div>

<Tabs queryString groupId="lang">
<TabItem value="py" label="Python"></TabItem>
<TabItem value="js" label="JavaScript"></TabItem>
</Tabs>

***

## Initiating a persistent Chroma client

<Tabs queryString groupId="lang" className="hideTabSwitcher">
<TabItem value="py" label="Python">

```python
import chromadb
```

You can configure Chroma to save and load from your local machine. Data will be persisted on exit and loaded on start (if it exists). This is useful for many experimental / prototyping workloads, limited by your machine's memory.

```python
from chromadb.config import Settings
client = chromadb.Client(Settings(
    chroma_db_impl="duckdb+parquet",
    persist_directory="/path/to/persist/directory" # Optional, defaults to .chromadb/ in the current directory
))
```

The `persist_directory` is where Chroma will store its database files on disk, and load them on start.

:::info Use a single client at-a-time
Having many in-memory clients that are loading and saving to the same path can cause strange behavior including data deletion. As a general practice, create an in-memory Chroma client once in your application, and pass it around instead of creating many clients.
:::
:::note Jupyter Notebooks
In a normal python program, `.persist()` will happening automatically if you set it. But in a Jupyter Notebook you will need to **manually** call `client.persist()`.
:::

</TabItem>
<TabItem value="js" label="JavaScript">

```js
import { ChromaClient } from 'chromadb'
```
:::note Connecting to the backend
To connect with the JS client, you must connect to a backend running Chroma. See `Running Chroma in client/server mode` for how to do this. 
:::

The JS client talks to a chroma server backend. This can run on your local computer via `docker` (see below) or be easily deployed to AWS.

```js
const client = new ChromaClient();
```

</TabItem>

</Tabs>


<Tabs queryString groupId="lang" className="hideTabSwitcher">
<TabItem value="py" label="Python">

The client object has a few useful convenience methods.

```python
client.heartbeat() # returns a nanosecond heartbeat. Useful for making sure the client remains connected.
client.reset() # Empties and completely resets the database. ⚠️ This is destructive and not reversible.
```

</TabItem>
<TabItem value="js" label="JavaScript">

The client object has a few useful convenience methods.

```javascript
await client.reset() # Empties and completely resets the database. ⚠️ This is destructive and not reversible.
```



</TabItem>

</Tabs>




## Running Chroma in client/server mode

<Tabs queryString groupId="lang" className="hideTabSwitcher">
<TabItem value="py" label="Python">

Chroma can also be configured to use an on-disk database, useful for larger data which doesn't fit in memory. To run Chroma in client server mode, run the docker container:

```bash
docker-compose up -d --build
```

Then update your chroma client to point at the docker container. Default: `localhost:8000`

```python
import chromadb
from chromadb.config import Settings
chroma_client = chromadb.Client(Settings(chroma_api_impl="rest",
                                        chroma_server_host="localhost",
                                        chroma_server_http_port="8000"
                                    ))
```

That's it! Chroma's API will run in `client-server` mode with just this change.


<!-- #### Run Chroma inside your application

To run the Chroma docker from inside your application code, create a docker-compose file or add to the existing one you have.

1. Download [`docker-compose.server.example.yml`](https://github.com/chroma-core/chroma/blob/main/docker-compose.server.example.yml) file and [`config`](https://github.com/chroma-core/chroma/tree/main/config) folder along with both the files inside from [GitHub Repo](https://github.com/chroma-core/chroma)
2. Rename `docker-compose.server.example.yml` to `docker-compose.yml`
3. Install docker on your local machine. [`Docker Engine`](https://docs.docker.com/engine/install/) or [`Docker Desktop`](https://docs.docker.com/desktop/install/)
4. Install docker compose [`Docker Compose`](https://docs.docker.com/compose/install/)

Use following command to manage Dockerized Chroma:
- __Command to Start Chroma__: `docker-compose up -d`
- __Command to Stop Chroma__: `docker-compose down`
- __Command to Stop Chroma and delete volumes__
This is distructive command. With this command volumes created earlier will be deleted along with data stored.: `docker-compose down -v` -->

#### Using the python http-only client

If you are running chroma in client-server mode. You may not require the full Chroma library and instead only the client library on your client machine. In this case, you can install the `chromadb-client` package. This package is a lightweight HTTP client for the server with a minimal dependency footprint.

```python
pip install chromadb-client
```

```python
import chromadb
from chromadb.config import Settings
# Example setup of the client to connect to your chroma server
client = chromadb.Client(Settings(chroma_api_impl="rest", chroma_server_host="localhost", chroma_server_http_port=8000))
```

Note that the `chromadb-client` package is a subset of the full Chroma library and does not include all the dependencies. If you want to use the full Chroma library, you can install the `chromadb` package instead. Most importantly, there is no default embedding function. If you add() documents without embeddings, you must have manually specified an embedding function and installed the dependencies for it.




</TabItem>
<TabItem value="js" label="JavaScript">

Chroma can be run in-memory in Python (without Docker), but this feature is not yet available in JS (this is on our roadmap). To run Chroma in client server mode, run the docker container:

```bash
docker-compose up -d --build
```

Then update your chroma client to point at the docker container. Default: `localhost:8000`

The JS client then talks to the chroma server backend. This can run on your local computer or be easily deployed to AWS.

```js
import { ChromaClient } from 'chromadb'
const client = new ChromaClient();
```

</TabItem>

</Tabs>


## Using collections

Chroma lets you manage collections of embeddings, using the `collection` primitive.

### Creating, inspecting, and deleting Collections

Chroma uses collection names in the url, so there are a few restrictions on naming them:

- The length of the name must be between 3 and 63 characters.
- The name must start and end with a lowercase letter or a digit, and it can contain dots, dashes, and underscores in between.
- The name must not contain two consecutive dots.
- The name must not be a valid IP address.

Chroma collections are created with a name and an optional embedding function. If you supply an embedding function, you must supply it every time you get the collection.

<Tabs queryString groupId="lang" className="hideTabSwitcher">
<TabItem value="py" label="Python">

```python
collection = client.create_collection(name="my_collection", embedding_function=emb_fn)
collection = client.get_collection(name="my_collection", embedding_function=emb_fn)
```

:::caution
If you later wish to `get_collection`, you MUST do so with the embedding function you supplied while creating the collection
:::

The embedding function takes text as input, and performs tokenization and embedding. If no embedding function is supplied, Chroma will use [sentence transfomer](https://www.sbert.net/index.html) as a default.

</TabItem>
<TabItem value="js" label="JavaScript">

```js
import { ChromaClient } from 'chromadb'
```

The JS client talks to a chroma server backend. This can run on your local computer or be easily deployed to AWS.

```js
let collection = await client.createCollection({name:"my_collection", embeddingFunction: emb_fn})
let collection2 = await client.getCollection({name:"my_collection", embeddingFunction: emb_fn})
```

:::caution
If you later wish to `getCollection`, you MUST do so with the embedding function you supplied while creating the collection
:::

The embedding function takes text as input, and performs tokenization and embedding. 

</TabItem>

</Tabs>


You can learn more about [🧬 embedding functions](./embeddings.md), and how to create your own.


<Tabs queryString groupId="lang" className="hideTabSwitcher">
<TabItem value="py" label="Python">

Existing collections can be retrieved by name with `.get_collection`, and deleted with `.delete_collection`. You can also use `.get_or_create_collection` to get a collection if it exists, or create it if it doesn't.


```python
collection = client.get_collection(name="test") # Get a collection object from an existing collection, by name. Will raise an exception if it's not found.
collection = client.get_or_create_collection(name="test") # Get a collection object from an existing collection, by name. If it doesn't exist, create it.
client.delete_collection(name="my_collection") # Delete a collection and all associated embeddings, documents, and metadata. ⚠️ This is destructive and not reversible
```

</TabItem>
<TabItem value="js" label="JavaScript">

Existing collections can be retrieved by name with `.getCollection`, and deleted with `.deleteCollection`.

```javascript
const collection = await client.getCollection({name: "test"}) # Get a collection object from an existing collection, by name. Will raise an exception of it's not found.
await client.deleteCollection({name: "my_collection"}) # Delete a collection and all associated embeddings, documents, and metadata. ⚠️ This is destructive and not reversible
```

</TabItem>

</Tabs>


Collections have a few useful convenience methods.

<Tabs queryString groupId="lang" className="hideTabSwitcher">
<TabItem value="py" label="Python">

```python
collection.peek() # returns a list of the first 10 items in the collection
collection.count() # returns the number of items in the collection
collection.modify(name="new_name") # Rename the collection
```

</TabItem>
<TabItem value="js" label="JavaScript">

```javascript
await collection.peek() // returns a list of the first 10 items in the collection
await collection.count() // returns the number of items in the collection
```

</TabItem>

</Tabs>

### Changing the distance function

<Tabs queryString groupId="lang" className="hideTabSwitcher">
<TabItem value="py" label="Python">

`create_collection` also takes an optional `metadata` argument which can be used to customize the distance method of the embedding space by setting the value of `hnsw:space`. 

```python
 collection = client.create_collection(
        name="collection_name",
        metadata={"hnsw:space": "cosine"} # l2 is the default
    )
```

</TabItem>
	
<TabItem value="js" label="Javascript">

`createCollection` also takes an optional `metadata` argument which can be used to customize the distance method of the embedding space by setting the value of `hnsw:space`

```js
 let collection = client.createCollection("collection_name", undefined, metadata={ "hnsw:space": "cosine" })
```

</TabItem>
	
</Tabs>

Valid options for `hnsw:space` are "l2", "ip, "or "cosine". The **default** is "l2". The equations for each can be found in the docs for Hnswlib [here](https://github.com/nmslib/hnswlib/tree/master#python-bindings).


### Adding data to a Collection

Add data to Chroma with `.add`.

Raw documents:

<Tabs queryString groupId="lang" className="hideTabSwitcher">
<TabItem value="py" label="Python">

```python
collection.add(
    documents=["lorem ipsum...", "doc2", "doc3", ...],
    metadatas=[{"chapter": "3", "verse": "16"}, {"chapter": "3", "verse": "5"}, {"chapter": "29", "verse": "11"}, ...],
    ids=["id1", "id2", "id3", ...]
)
```

</TabItem>
<TabItem value="js" label="JavaScript">

```javascript
await collection.add({
    ids: ["id1", "id2", "id3", ...],
    metadatas: [{"chapter": "3", "verse": "16"}, {"chapter": "3", "verse": "5"}, {"chapter": "29", "verse": "11"}, ...], 
    documents: ["lorem ipsum...", "doc2", "doc3", ...], 
})
// input order
// ids - required
// embeddings - optional
// metadata - optional
// documents - optional
```

</TabItem>

</Tabs>



If Chroma is passed a list of `documents`, it will automatically tokenize and embed them with the collection's embedding function (the default will be used if none was supplied at collection creation). Chroma will also store the `documents` themselves. If the documents are too large to embed using the chosen embedding function, an exception will be raised.

Each document must have a unique associated `id`. Trying to `.add` the same ID twice will result in an error. An optional list of `metadata` dictionaries can be supplied for each document, to store additional information and enable filtering.

Alternatively, you can supply a list of document-associated `embeddings` directly, and Chroma will store the associated documents without embedding them itself.

<Tabs queryString groupId="lang" className="hideTabSwitcher">
<TabItem value="py" label="Python">

```python
collection.add(
    documents=["doc1", "doc2", "doc3", ...],
    embeddings=[[1.1, 2.3, 3.2], [4.5, 6.9, 4.4], [1.1, 2.3, 3.2], ...],
    metadatas=[{"chapter": "3", "verse": "16"}, {"chapter": "3", "verse": "5"}, {"chapter": "29", "verse": "11"}, ...],
    ids=["id1", "id2", "id3", ...]
)
```

</TabItem>
<TabItem value="js" label="JavaScript">

```javascript
await collection.add({
    ids: ["id1", "id2", "id3", ...], 
    embeddings: [[1.1, 2.3, 3.2], [4.5, 6.9, 4.4], [1.1, 2.3, 3.2], ...],
    metadatas: [{"chapter": "3", "verse": "16"}, {"chapter": "3", "verse": "5"}, {"chapter": "29", "verse": "11"}, ...], 
    documents: ["lorem ipsum...", "doc2", "doc3", ...], 
})

```

</TabItem>

</Tabs>



If the supplied `embeddings` are not the same dimension as the collection, an exception will be raised.

You can also store documents elsewhere, and just supply a list of `embeddings` and `metadata` to Chroma. You can use the `ids` to associate the embeddings with your documents stored elsewhere.

<Tabs queryString groupId="lang" className="hideTabSwitcher">
<TabItem value="py" label="Python">

```python
collection.add(
    embeddings=[[1.1, 2.3, 3.2], [4.5, 6.9, 4.4], [1.1, 2.3, 3.2], ...],
    metadatas=[{"chapter": "3", "verse": "16"}, {"chapter": "3", "verse": "5"}, {"chapter": "29", "verse": "11"}, ...],
    ids=["id1", "id2", "id3", ...]
)
```

</TabItem>
<TabItem value="js" label="JavaScript">

```javascript
await collection.add({
    ids: ["id1", "id2", "id3", ...], 
    embeddings: [[1.1, 2.3, 3.2], [4.5, 6.9, 4.4], [1.1, 2.3, 3.2], ...], 
    metadatas: [{"chapter": "3", "verse": "16"}, {"chapter": "3", "verse": "5"}, {"chapter": "29", "verse": "11"}, ...], 
})
```

</TabItem>

</Tabs>



### Querying a Collection

Chroma collections can be queried in a variety of ways, using the `.query` method.

You can query by a set of `query_embeddings`.

<Tabs queryString groupId="lang" className="hideTabSwitcher">
<TabItem value="py" label="Python">

```python
collection.query(
    query_embeddings=[[11.1, 12.1, 13.1],[1.1, 2.3, 3.2], ...],
    n_results=10,
    where={"metadata_field": "is_equal_to_this"},
    where_document={"$contains":"search_string"}
)
```

</TabItem>
<TabItem value="js" label="JavaScript">

```javascript
const result = await collection.query({
    queryEmbeddings: [[11.1, 12.1, 13.1],[1.1, 2.3, 3.2], ...],
    nResults: 10, 
    where: {"metadata_field": "is_equal_to_this"}, 
})
// input order
// query_embeddings - optional
// n_results - required
// where - optional
// query_texts - optional
```

</TabItem>

</Tabs>



The query will return the `n_results` closest matches to each `query_embedding`, in order.
An optional `where` filter dictionary can be supplied to filter the results by the `metadata` associated with each document.
Additionally, an optional `where_document` filter dictionary can be supplied to filter the results by contents of the document.

If the supplied `query_embeddings` are not the same dimension as the collection, an exception will be raised.

You can also query by a set of `query_texts`. Chroma will first embed each `query_text` with the collection's embedding function, and then perform the query with the generated embedding.

<Tabs queryString groupId="lang" className="hideTabSwitcher">
<TabItem value="py" label="Python">

```python
collection.query(
    query_texts=["doc10", "thus spake zarathustra", ...],
    n_results=10,
    where={"metadata_field": "is_equal_to_this"},
    where_document={"$contains":"search_string"}
)
```

You can also retrieve items from a collection by `id` using `.get`.

```python
collection.get(
	ids=["id1", "id2", "id3", ...],
	where={"style": "style1"}
)
```

</TabItem>
<TabItem value="js" label="JavaScript">

```javascript
await collection.query({
    nResults: 10, // n_results
    where: {"metadata_field": "is_equal_to_this"}, // where
    queryTexts: ["doc10", "thus spake zarathustra", ...], // query_text
})
```

You can also retrieve items from a collection by `id` using `.get`.

```javascript
await collection.get({
	ids: ["id1", "id2", "id3", ...], //ids
	where: {"style": "style1"} // where
})
```

</TabItem>

</Tabs>



`.get` also supports the `where` and `where_document` filters. If no `ids` are supplied, it will return all items in the collection that match the `where` and `where_document` filters.

##### Choosing which data is returned
When using get or query you can use the include parameter to specify which data you want returned - any of `embeddings`, `documents`, `metadatas`, and for query, `distances`. By default, Chroma will return the `documents`, `metadatas` and in the case of query, the `distances` of the results. `embeddings` are excluded by default for performance and the `ids` are always returned. You can specify which of these you want returned by passing an array of included field names to the includes parameter of the query or get method.

```python

# Only get documents and ids
collection.get(
    include=["documents"]
)

collection.query(
    query_embeddings=[[11.1, 12.1, 13.1],[1.1, 2.3, 3.2], ...],
    include=["documents"]
)
```

### Using Where filters

Chroma supports filtering queries by `metadata` and `document` contents. The `where` filter is used to filter by `metadata`, and the `where_document` filter is used to filter by `document` contents.

##### Filtering by metadata
In order to filter on metadata, you must supply a `where` filter dictionary to the query. The dictionary must have the following structure:

```python
{
    "metadata_field": {
        <Operator>: <Value>
    }
}
```

Filtering metadata supports the following operators:

- `$eq` - equal to (string, int, float)
- `$ne` - not equal to (string, int, float)
- `$gt` - greater than (int, float)
- `$gte` - greater than or equal to (int, float)
- `$lt` - less than (int, float)
- `$lte` - less than or equal to (int, float)

Using the $eq operator is equivalent to using the `where` filter.
```python
{
    "metadata_field": "search_string"
}

# is equivalent to

{
    "metadata_field": {
        "$eq": "search_string"
    }
}

```
##### Filtering by document contents
In order to filter on document contents, you must supply a `where_document` filter dictionary to the query. The dictionary must have the following structure:

```python
# Filtering for a search_string
{
    "$contains": "search_string"
}
``` 


##### Using logical operators

You can also use the logical operators `$and` and `$or` to combine multiple filters.

An `$and` operator will return results that match all of the filters in the list.
```python
{
    "$and": [
        {
            "metadata_field": {
                <Operator>: <Value>
            }
        },
        {
            "metadata_field": {
                <Operator>: <Value>
            }
        }
    ]
}
```

An `$or` operator will return results that match any of the filters in the list.
```python
{
    "$or": [
        {
            "metadata_field": {
                <Operator>: <Value>
            }
        },
        {
            "metadata_field": {
                <Operator>: <Value>
            }
        }
    ]
}
```

<Tabs queryString groupId="lang" className="hideTabSwitcher">
<TabItem value="py" label="Python">

### Updating data in a collection

Any property of items in a collection can be updated using `.update`.

```python
collection.update(
    ids=["id1", "id2", "id3", ...],
    embeddings=[[1.1, 2.3, 3.2], [4.5, 6.9, 4.4], [1.1, 2.3, 3.2], ...],
    metadatas=[{"chapter": "3", "verse": "16"}, {"chapter": "3", "verse": "5"}, {"chapter": "29", "verse": "11"}, ...],
    documents=["doc1", "doc2", "doc3", ...],
)
```

</TabItem>
<TabItem value="js" label="JavaScript">

</TabItem>

</Tabs>



If an `id` is not found in the collection, an exception will be raised. If `documents` are supplied without corresponding `embeddings`, the embeddings will be recomupted with the collection's embedding function.

If the supplied `embeddings` are not the same dimension as the collection, an exception will be raised.

Chroma also supports an `upsert` operation, which updates existing items, or adds them if they don't yet exist.

<Tabs queryString groupId="lang" className="hideTabSwitcher">
<TabItem value="py" label="Python">

```python
collection.upsert(
    ids=["id1", "id2", "id3", ...],
    embeddings=[[1.1, 2.3, 3.2], [4.5, 6.9, 4.4], [1.1, 2.3, 3.2], ...],
    metadatas=[{"chapter": "3", "verse": "16"}, {"chapter": "3", "verse": "5"}, {"chapter": "29", "verse": "11"}, ...],
    documents=["doc1", "doc2", "doc3", ...],
)
```

</TabItem>
<TabItem value="js" label="JavaScript">

```javascript
await collection.upsert({
    ids: ["id1", "id2", "id3"],
    embeddings: [[1.1, 2.3, 3.2], [4.5, 6.9, 4.4], [1.1, 2.3, 3.2]],
    metadatas: [{"chapter": "3", "verse": "16"}, {"chapter": "3", "verse": "5"}, {"chapter": "29", "verse": "11"}],
    documents: ["doc1", "doc2", "doc3"]
})
```
</TabItem>
</Tabs>

If an `id` is not present in the collection, the corresponding items will be created as per `add`. Items with existing `id`s will be updated as per `update`.

### Deleting data from a collection

Chroma supports deleting items from a collection by `id` using `.delete`. The embeddings, documents, and metadata associated with each item will be deleted.
⚠️ Naturally, this is a destructive operation, and cannot be undone.

<Tabs queryString groupId="lang" className="hideTabSwitcher">
<TabItem value="py" label="Python">

```python
collection.delete(
    ids=["id1", "id2", "id3",...],
	where={"chapter": "20"}
)
```

</TabItem>
<TabItem value="js" label="JavaScript">

```javascript
await collection.delete({
    ids: ["id1", "id2", "id3",...], //ids
	where: {"chapter": "20"} //where
})
```

</TabItem>

</Tabs>



`.delete` also supports the `where` filter. If no `ids` are supplied, it will delete all items in the collection that match the `where` filter.
