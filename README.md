- [Get Started With Langchain and Pinecone in Node.js](#get-started-with-langchain-and-pinecone-in-nodejs)
  - [Back to the tutorial](#back-to-the-tutorial)
  - [References](#references)
  - [Versions](#versions)
  - [File Loaders](#file-loaders)
  - [Understanding indexes](#understanding-indexes)
  - [Pods](#pods)
  - [Vector databases](#vector-databases)
  - [Understanding collections](#understanding-collections)
  - [Computing the vector dimension](#computing-the-vector-dimension)
    - [Number of vectors](#number-of-vectors)
    - [Dimensionality of vectors](#dimensionality-of-vectors)
- [0-main.js](#0-mainjs)
  - [1-createPineconeIndex.js](#1-createpineconeindexjs)
  - [Execution](#execution)
  - [Pinecone Indexes Dashboard](#pinecone-indexes-dashboard)

# Get Started With Langchain and Pinecone in Node.js

## Back to the tutorial

You are in lesson [Pinecone](https://github.com/ULL-prompt-engineering/prompt-engineering-101/tree/main#lesson-7-pinecone) of the [Prompt Engineering 101](https://github.com/ULL-prompt-engineering/prompt-engineering-101/blob/main/README.md) course.

## References

* See Developers Digest youtube tutorial: https://youtu.be/CF5buEVrYwo. This crash course will guide us through the basics of programming with Langchain and Pinecone.
  * [![Langchain and Pinecone Youtube tutorial](https://img.youtube.com/vi/CF5buEVrYwo/maxresdefault.jpg)](https://youtu.be/CF5buEVrYwo) 
* [Prompt Engineering 101](https://github.com/ULL-prompt-engineering/prompt-engineering-101/blob/main/README.md) course
* [What is a Vector Database?](https://www.pinecone.io/learn/vector-database/) by Roie Schwaber-Cohen
* [Google Developers Machine Learning Crash Course: Embeddings video](https://developers.google.com/machine-learning/crash-course/embeddings/video-lecture?hl=en)


## Versions

Use `nvm` to install these versions of node and npm:

```bash
➜  Get-Started-With-Langchain-and-Pinecone-in-Node.js git:(casiano) ✗ nvm use v18.16
Now using node v18.16.1 (npm v9.5.1)
➜  Get-Started-With-Langchain-and-Pinecone-in-Node.js git:(casiano) ✗ npm --version
9.5.1
```

## File Loaders

See 

* Section LangChain [Document Loaders](https://github.com/ULL-prompt-engineering/prompt-engineering-101/tree/main/lesson5#document-loaders) in the [Prompt Engineering 101](https://github.com/ULL-prompt-engineering/prompt-engineering-101/blob/main/README.md) course
  * Example [document-pdf-loader.mjs](https://github.com/ULL-prompt-engineering/prompt-engineering-101/blob/main/lesson5/src/document-pdf-loader.mjs)
  * Example [document-github-loader.mjs](https://github.com/ULL-prompt-engineering/prompt-engineering-101/blob/main/lesson5/src/document-github-loader.mjs)
* [LangChain File Loaders](https://js.langchain.com/docs/modules/data_connection/document_loaders/integrations/file_loaders/) in LangChain docs

## Understanding indexes

Standalone vector indices like [FAISS](https://www.pinecone.io/learn/faiss/) (**Facebook AI Similarity Search**) can significantly **improve search and retrieval of vector embeddings** (but they lack capabilities that exist in a database). 

An index is the highest-level organizational unit of vector data in Pinecone. 
It accepts and stores vectors, **serves queries over the vectors it contains**, and does other vector operations over its contents. 

Each index runs on at least one **pod**[^1].

[^1]: **seed pod**. noun. a **carpel** (female reproductive organ) or **pistil** (usually is located in the center of the flower is made up of three parts: the stigma, style, and ovary) enclosing the seeds of a plant, esp a flowering plant. So **pineconce pods** are the seeds of the pinecone.

## Pods

**Pods** are pre-configured units of hardware for running a Pinecone service. 

Each index runs on one or more pods. Generally, more pods mean more storage capacity, lower latency, and higher throughput. You can also create pods of different sizes.

Once an index is created using a particular pod type, you cannot change the pod type for that index. However, [you can create a new index from that collection](https://docs.pinecone.io/docs/manage-indexes/#create-an-index-from-a-collection) with a different pod type. 

Different pod types are priced differently. See [pricing](https://www.pinecone.io/pricing/) for more details.

## Vector databases

With a vector database, we can add advanced features to our AIs, like 

1. semantic information retrieval, 
2. long-term memory. 

Vector databases are built to manage vector embeddings, providing several advantages over using standalone vector indices:

1. **Data management**: Vector databases offer well-known and easy-to-use features for data storage, like inserting, deleting, and updating data
2. **Metadata storage and filtering**: Vector databases can store metadata associated with each vector entry. Users can then query the database using additional metadata filters for finer-grained queries
3. **Scalability**
4. **Real-time updates**: Vector indexes may require a full re-indexing process to incorporate new data, which can be time-consuming and computationally expensive.
5. **Backups and collections**: Pinecone also allows users to selectively choose specific indexes that can be backed up in the form of **“collections**,” which store the data in that index for later use.
6. **Ecosystem integration:** Vector databases can more easily integrate with other components of a data processing ecosystem, such as
   1.  ETL pipelines[^2] (like [Spark](https://spark.apache.org/)), 
   2.  analytics tools (like [Tableau](https://www.tableau.com/) and [Segment](https://segment.com/)), and 
   3.  visualization platforms (like [Grafana](https://grafana.com/)) – streamlining the data management workflow. It also enables easy integration with other AI related tools like 
   4.  [LangChain](https://python.langchain.com/en/latest/index.html), 
   5.  [LlamaIndex](https://gpt-index.readthedocs.io/) and 
   6.  [ChatGPT’s Plugins](https://openai.com/blog/chatgpt-plugins)..
7. **Data security and access control**

[^2]: An ETL pipeline is the set of processes used to move data from a source or multiple sources into a database such as a data warehouse. **ETL stands for “extract, transform, load,”** the three interdependent processes of data integration used to pull data from one database and move it to another.

## Understanding collections

A **collection** is a static copy of an index. It is a non-queryable representation of a set of vectors and metadata. You can create a collection from an index, and you can create a new index from a collection. This new index can differ from the original source index: the new index can have a different number of pods, a different pod type, or a different similarity metric.

Creating a collection from your index is useful when performing tasks like the following:

- Temporarily shutting down an index
- Copying the data from one index into a different index;
- Making a backup of your index
- Experimenting with different index configurations

## Computing the vector dimension

**Adjusting the vector dimension can impact the accuracy and performance of the search, so it's important to choose an adequate dimension based on your specific use case and data.**

In the example [0-main.js](0-main.js) we have:

```js
const vectorDimension = 1536;
```

and in [1-createPineconeindex.js](1-createPineconeindex.js) we have:

```js
// 5. Create index
    const createClient = await client.createIndex({
      createRequest: {
        name: indexName,
        dimension: vectorDimension,
        metric: "cosine",
      },
    });
```

The presenter says:

> Since we're using the OpenAI embeddings endpoint, this is the criteria that's returned from the embeddings endpoint. If you are using a different embeddings endpoint, you'll need to set up the proper vector dimension accordingly.


OpenAI’s text embeddings measure the relatedness of text strings. **Embeddings** are commonly used for:

- Search (where results are ranked by relevance to a query string)
- Clustering (where text strings are grouped by similarity)
- Recommendations (where items with related text strings are recommended)
- Anomaly detection (where outliers with little relatedness are identified)
- Diversity measurement (where similarity distributions are analyzed)
- Classification (where text strings are classified by their most similar label)

An embedding is a vector (list) of floating point numbers. The distance between two vectors measures their relatedness. Small distances suggest high relatedness and large distances suggest low relatedness.

If you go to [Embeddings reference guide](https://platform.openai.com/docs/guides/embeddings/embedding-models) you can see their recommendations.

<table>
<thead>
<tr>
<th>Model name</th><th>tokenizer</th><th>max input tokens</th><th>output dimensions</th>
</tr>
</thead>
<tbody>
<tr><td>text-embedding-ada-002</td><td>cl100k_base</td><td>8191</td><td>1536</td></tr>
</tbody>
</table>

See section [Choosing index type and size](https://docs.pinecone.io/docs/choosing-index-type-and-size) of  the Pinecone docs.

There are five main considerations when deciding how to configure your Pinecone index:

- Number of vectors
- Dimensionality of your vectors
- Size of metadata on each vector
- [Queries per second (QPS) throughput](https://docs.pinecone.io/docs/choosing-index-type-and-size#queries-per-second-qps)
- [Cardinality of indexed metadata](https://docs.pinecone.io/docs/choosing-index-type-and-size#metadata-cardinality-and-size)

### Number of vectors

The most important consideration in sizing is the number of vectors you plan on working with. As a rule of thumb, a single p1 pod can store approximately 1M vectors, while a s1 pod can store 5M vectors. However, this can be affected by other factors, such as dimensionality and metadata.

### Dimensionality of vectors

By specifying the vector dimension when creating the PineconeStore object, you can control the dimensionality of the vectors used for similarity search. 

The rules of thumb above assumes a typical configuration of 768 [dimensions per vector](https://docs.pinecone.io/docs/manage-indexes/#creating-an-index). As your individual use case will dictate the dimensionality of your vectors, the amount of space required to store them may necessarily be larger or smaller.

Each dimension on a single vector consumes 4 bytes of memory and storage per dimension, so if you expect to have 1M vectors with 768 dimensions each, that’s about 3GB of storage without factoring in metadata or other overhead. 

Using that reference, we can estimate the typical pod size and number needed for a given index. This [Table](https://docs.pinecone.io/docs/choosing-index-type-and-size#dimensionality-of-vectors)  gives some examples of this.

A good empirical rule of thumb is [the number of dimensions to be **roughly the fourth root of the size of my vocabulary**, the number of possible values](https://developers.google.com/machine-learning/crash-course/embeddings/video-lecture?hl=en). But this is just a rule of thumb and with all hyperparameters you really need to go use validation data and try it out for your problem and see what gives the best results.

# 0-main.js

The first part is:

1. Initialize a new project with: `npm init -y` or use the [package.json](package.json)
2.  Create the four empty `.js` files 
   - [0-main.js](0-main.js)                     
   - [1-createPineconeIndex.js](1-createPineconeIndex.js)      
   - [2-updatePinecone.js](2-updatePinecone.js)           
   - [3-queryPineconeAndQueryGPT.js](3-queryPineconeAndQueryGPT.js)
3. Create the [.env](.env) file 
4. Install depedencies: `npm i "@pinecone-database/pinecone@^0.0.10" dotenv@^16.0.3 langchain@^0.0.73` or `npm i` if you are reusing the [package.json](package.json)
5. Obtain API key from [OpenAI](https://platform.openai.com/account/api-keys) (https://platform.openai.com/account/api-keys)
6. Obtain API key from [Pinecone](https://app.pinecone.io/) (https://app.pinecone.io/)

   [![Obtain API key from Pinecone](/images/pinecone-api-key.png)](https://app.pinecone.io/)
7. Fill the API keys in `.env` file

Now we make our imports:

```js
import { PineconeClient } from "@pinecone-database/pinecone";
import { DirectoryLoader } from "langchain/document_loaders/fs/directory";
import { TextLoader } from "langchain/document_loaders/fs/text";
import { PDFLoader } from "langchain/document_loaders/fs/pdf";
import * as dotenv from "dotenv";
import { createPineconeIndex } from "./1-createPineconeIndex.js";
import { updatePinecone } from "./2-updatePinecone.js";
import { queryPineconeVectorStoreAndQueryLLM } from "./3-queryPineconeAndQueryGPT.js";
```
Optionally, if we want to use other file loaders for CSV, JSON, etc. (https://js.langchain.com/docs/modules/indexes/document_loaders/examples/file_loaders/) we can import them as well.

Then we load the environment variables from `.env`  that will be accesible in `process.env`:

```js 
dotenv.config();
```

Set up `DirectoryLoader` to load all the documents from the `./documents` directory

```js
const loader = new DirectoryLoader("./documents", {
    ".txt": (path) => new TextLoader(path),
    ".pdf": (path) => new PDFLoader(path),
});
const docs = await loader.load();
```
The [DirectoryLoader](https://js.langchain.com/docs/api/document_loaders_fs_directory/classes/DirectoryLoader) is a class that allows you to load all documents in a directory. It takes two arguments: the `directory` path and a map of file extensions to loader factories. Each file in the `directory` will be passed to the matching loader, and the resulting documents will be concatenated together.

Next, we set up variables for the `question`, the Pinecone `indexName`, and the `vectorDimension`.
Adjusting the vector dimension can impact the accuracy and performance of the search, so it's important to choose an adequate dimension based on your specific use case and data.

```js
const question = "What is the most hidden secret?";
const indexName = "your-pinecone-index-name";
const vectorDimension = 1536;
```
A good empirical rule of thumb is [the number of dimensions to be **roughly the fourth root of the size of my vocabulary**, the number of possible values](https://developers.google.com/machine-learning/crash-course/embeddings/video-lecture?hl=en). But this is just a rule of thumb and with all hyperparameters you really need to go use validation data and try it out for your problem and see what gives the best results. If you go to the [OpenAI Embeddings reference guide](https://platform.openai.com/docs/guides/embeddings/embedding-models) you can see among their recommendations this:

<table>
<thead>
<tr>
<th>Model name</th><th>tokenizer</th><th>max input tokens</th><th>output dimensions</th>
</tr>
</thead>
<tbody>
<tr><td>text-embedding-ada-002</td><td>cl100k_base</td><td>8191</td><td>1536</td></tr>
</tbody>
</table>

See section [Choosing index type and size](https://docs.pinecone.io/docs/choosing-index-type-and-size) of  the Pinecone docs.

Now we initialize the Pinecone client with the API key and the environment. The environment is the string (s.t. like `us-west1-gcp-free`) you see  in the [Pinecone console](https://app.pinecone.io/). If you don't specify an environment, the client will use the default environment.

```js
const client = new PineconeClient();
await client.init({
  apiKey: process.env.PINECONE_API_KEY,
  environment: process.env.PINECONE_ENVIRONMENT,
});
```
We then  run the  `async` function containing the main code.
1. The function `createPineconeIndex` exported by `1-createPineconeIndex.js` checks if the index exists and creates it if necessary. 
2. The function `updatePinecone` exported by `2-updatePineconce.js` updates the Pinecone vector store with document embeddings. 
3. The function `queryPineconeVectorStoreAndQueryLLM` exported by `3-queryPineconceAndQueryGPT` queries the Pinecone vector store and GPT model for an answer.

```js

```js
(async () => {
  await createPineconeIndex(client, indexName, vectorDimension);
  await updatePinecone(client, indexName, docs);
  await queryPineconeVectorStoreAndQueryLLM(client, indexName, question);
})();
```

## 1-createPineconeIndex.js

```js
export const createPineconeIndex = async (client, indexName, vectorDimension) => {
  // 1. Initiate index existence check
  console.log(`Checking "${indexName}"...`);
  // 2. Get list of existing indexes
  const existingIndexes = await client.listIndexes();

  // 3. If index doesn't exist, create it
  if (!existingIndexes.includes(indexName)) {
    // 4. Log index creation initiation
    console.log(`Creating "${indexName}"...`);
    // 5. Create index
    const createClient = await client.createIndex({
      createRequest: {
        name: indexName,
        dimension: vectorDimension,
        metric: "cosine",
      },
    });
    // 6. Log successful creation
    console.log(`Created with client:`, createClient);
    
    // 7. Wait 60 seconds for index initialization
    await new Promise((resolve) => setTimeout(resolve, 60000));
  
  } else {
    // 8. Log if index already exists
    console.log(`"${indexName}" already exists.`);
  }
};
```

## Execution

``` 
➜  Get-Started-With-Langchain-and-Pinecone-in-Node.js git:(casiano) node 0-main.js
Checking "your-pinecone-index-name"...
"your-pinecone-index-name" already exists.
Retrieving Pinecone index...
Pinecone index retrieved: your-pinecone-index-name
Processing document: /Users/casianorodriguezleon/campus-virtual/2223/learning/pinecone-learning/Get-Started-With-Langchain-and-Pinecone-in-Node.js/documents/Romeo and Juliet.txt
Splitting text into chunks...
Text split into 216 chunks
Calling OpenAI's Embedding endpoint documents with 216 text chunks ...
Finished embedding documents
Creating 216 vectors array with id, values, and metadata...
Upserted undefined vectors
Upserted undefined vectors
Upserted undefined vectors
Pinecone index updated with 216 vectors
Processing document: /Users/casianorodriguezleon/campus-virtual/2223/learning/pinecone-learning/Get-Started-With-Langchain-and-Pinecone-in-Node.js/documents/The Great Gatsby.txt
Splitting text into chunks...
Text split into 382 chunks
Calling OpenAI's Embedding endpoint documents with 382 text chunks ...
Finished embedding documents
Creating 382 vectors array with id, values, and metadata...
Upserted undefined vectors
Upserted undefined vectors
Upserted undefined vectors
Upserted undefined vectors
Pinecone index updated with 382 vectors
Querying Pinecone vector store...
Found 10 matches...
Asking question: What is the most hidden secret?...
Answer:  The most hidden secret is that Wilson is gay.
```

## Pinecone Indexes Dashboard

This is the `indexes` dashboard:

![/images/pinecone-index-dashboard-1.png](/images/pinecone-index-dashboard-1.png)

When we click on the `your-pinecone-index-name` link we can see:

![/images/pinecone-index-dashboard-2.png](/images/pinecone-index-dashboard-2.png)

When we click on the `connect` button we can see a pop-up window with the code:

![/images/pinecone-index-dashboard-3.png](/images/pinecone-index-dashboard-3.png)
