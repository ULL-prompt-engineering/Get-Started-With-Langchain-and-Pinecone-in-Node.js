# Get-Started-With-Langchain-and-Pinecone-in-Node.js

From my youtube tutorial: https://youtu.be/CF5buEVrYwo

## Versions

```bash
➜  Get-Started-With-Langchain-and-Pinecone-in-Node.js git:(casiano) ✗ nvm use v18.16
Now using node v18.16.1 (npm v9.5.1)
➜  Get-Started-With-Langchain-and-Pinecone-in-Node.js git:(casiano) ✗ npm --version
9.5.1
```

## File Loaders

* [LangChain File Loaders](https://js.langchain.com/docs/modules/data_connection/document_loaders/integrations/file_loaders/)

## Understanding indexes

An index is the highest-level organizational unit of vector data in Pinecone. 

It accepts and stores vectors, serves queries over the vectors it contains, and does other vector operations over its contents. 

Each index runs on at least one pod.

Pods are pre-configured units of hardware for running a Pinecone service. 

Each index runs on one or more pods. Generally, more pods mean more storage capacity, lower latency, and higher throughput. You can also create pods of different sizes.

Once an index is created using a particular pod type, you cannot change the pod type for that index. However, [you can create a new index from that collection](https://docs.pinecone.io/docs/manage-indexes/#create-an-index-from-a-collection) with a different pod type. 


Different pod types are priced differently. See [pricing](https://www.pinecone.io/pricing/) for more details.

## Understanding collections

A **collection** is a static copy of an index. It is a non-queryable representation of a set of vectors and metadata. You can create a collection from an index, and you can create a new index from a collection. This new index can differ from the original source index: the new index can have a different number of pods, a different pod type, or a different similarity metric.

Creating a collection from your index is useful when performing tasks like the following:

- Temporarily shutting down an index
- Copying the data from one index into a different index;
- Making a backup of your index
- Experimenting with different index configurations

## Computing the vector dimension

**Adjusting the vector dimension can impact the accuracy and performance of the search, so it's important to choose an adequate dimension based on your specific use case and data.**

See section [Choosing index type and size](https://docs.pinecone.io/docs/choosing-index-type-and-size) of  the Pinecone docs.

There are five main considerations when deciding how to configure your Pinecone index:

- Number of vectors
- Dimensionality of your vectors
- Size of metadata on each vector
- Queries per second (QPS) throughput
- Cardinality of indexed metadata

### Number of vectors

The most important consideration in sizing is the number of vectors you plan on working with. As a rule of thumb, a single p1 pod can store approximately 1M vectors, while a s1 pod can store 5M vectors. However, this can be affected by other factors, such as dimensionality and metadata.

### Dimensionality of vectors

The rules of thumb above assumes a typical configuration of 768 [dimensions per vector](https://docs.pinecone.io/docs/manage-indexes/#creating-an-index). As your individual use case will dictate the dimensionality of your vectors, the amount of space required to store them may necessarily be larger or smaller.

Each dimension on a single vector consumes 4 bytes of memory and storage per dimension, so if you expect to have 1M vectors with 768 dimensions each, that’s about 3GB of storage without factoring in metadata or other overhead. 

Using that reference, we can estimate the typical pod size and number needed for a given index. This [Table](https://docs.pinecone.io/docs/choosing-index-type-and-size#dimensionality-of-vectors)  gives some examples of this.

### Example

Import the necessary libraries and initialize the Pinecone client:

```js
import { PineconeClient } from "@pinecone-database/pinecone";
import * as dotenv from "dotenv";

dotenv.config();

const client = new PineconeClient();
await client.init({
  apiKey: process.env.PINECONE_API_KEY,
  environment: process.env.PINECONE_ENVIRONMENT,
});
const pineconeIndex = client.Index(process.env.PINECONE_INDEX);
Create a PineconeStore object with the desired vector dimension:
const vectorDimension = 128; // Replace with your desired vector dimension
const pineconeStore = new PineconeStore(vectorDimension, { pineconeIndex });
```

Add your documents to the PineconeStore:

```js
const docs = [
  new Document({
    metadata: { foo: "bar" },
    pageContent: "pinecone is a vector db",
  }),
  // Add more documents as needed
];

await pineconeStore.addDocuments(docs);
Compute the similarity search using the desired vector dimension:
const query = "pinecone"; // Replace with your query
const k = 10; // Replace with the number of nearest neighbors you want to retrieve

const results = await pineconeStore.similaritySearch(query, k);
console.log(results);
```
By specifying the vector dimension when creating the PineconeStore object, you can control the dimensionality of the vectors used for similarity search. 


