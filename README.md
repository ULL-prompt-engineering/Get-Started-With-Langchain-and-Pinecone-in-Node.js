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
  - [2-updatePinecone.js](#2-updatepineconejs)
  - [Execution](#execution)
  - [Pinecone Indexes Dashboard](#pinecone-indexes-dashboard)

# Get Started With Langchain and Pinecone in Node.js

When you have a custom knowledebase (KB), the first order of business is translating every piece of information into a vector - a multi-dimensional mathematical representation of that information

Vector databases help you store and look up these vectors efficiently. Fundamentally, they enable you to find similar items based on a query vector. So you can easily perform searches for "like" or "close to" other words or phrases, at scale.

LLMs like GPT4, don't know  information from your custom knowledgebase, when a user sends a prompt/query, we first have to look-up the Vector database for the relevant piece of information

For example, if you are looking for information to log-in to your internal HR system, and you ask your custom ChatBot, it needs to go to the vector database to first fetch the web page that has this information 

The custom ChatBot will then send this information to the LLM API (GPT-4) along with the user-query. The LLM API will then answer the user's question by reading the webpage about your internal HR system

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

[The `init` method of a `PineconeClient` object](https://docs.pinecone.io/docs/node-client#init) is used to initialize the client and establish a connection with the Pinecone service. It takes an object as a parameter with the following properties:

- `apiKey`: This property is used to provide the API key for authentication with the Pinecone service. The API key is obtained from the Pinecone dashboard and is required to access the service.
- `environment`: This property is used to specify the cloud environment of your Pinecone project.

We then  run the  `async` function containing the main code that takes three steps:

1. The function `createPineconeIndex` exported by `1-createPineconeIndex.js` checks if the index exists and creates it if necessary. 
2. The function `updatePinecone` exported by `2-updatePineconce.js` updates the Pinecone vector store with document embeddings. 
3. The function `queryPineconeVectorStoreAndQueryLLM` exported by `3-queryPineconceAndQueryGPT` queries the Pinecone vector store and GPT model for an answer.

```js
(async () => {
  await createPineconeIndex(client, indexName, vectorDimension);
  await updatePinecone(client, indexName, docs);
  await queryPineconeVectorStoreAndQueryLLM(client, indexName, question);
})();
```

## 1-createPineconeIndex.js


The function `createPineconeIndex`  is called with the Pinecone client, the name of the Pinecone index and the vector dimension 
 `await createPineconeIndex(client, indexName, vectorDimension)` and checks if the index exists and creates it if necessary.

To check for the existence we get list of existing indexes using [client.listIndexes()](https://docs.pinecone.io/docs/node-client#listindexes) that returns an array of strings with the names of the indexes in our project.
 
``` js
export const createPineconeIndex = async (client, indexName, vectorDimension) => {
  const existingIndexes = await client.listIndexes();

  if (!existingIndexes.includes(indexName)) {
    // Create index ...
    // Wait XX seconds for index initialization  
  } else {
    console.log(`"${indexName}" already exists.`);
  }
};

```
To create an index we use the method [client.createIndex](https://docs.pinecone.io/docs/node-client#createindex). 
By default, Pinecone indexes all metadata but you can specify which metadata fields to index by passing a list of field names to the `indexed` property of the `createRequest.metadataConfig` object. The following example creates an index that only indexes the "`color`" metadata field. Queries against this index cannot filter based on any other metadata field.

```js
await client.createIndex({
  createRequest: {
    name: "example-index-2",
    dimension: 1024,
    metadataConfig: {
      indexed: ["color"],
    },
  },
});
```

The following table describes the parameters of the `createIndex` method:

| Parameters | Type    | Description                                                                                                                                                                                                                                                                                                                                                   |
| ----------|---------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `name`     | str     | The name of the index to be created. The maximum length is 45 characters                                                                                                                                                                                                                                                                                      |
| `dimension`| integer | The dimensions of the vectors to be inserted in the index.                                                                                                                                                                                                                                                                                                    |
| `metric`   | str     | (Optional) The distance metric to be used for similarity search: 'euclidean', 'cosine', or 'dotproduct'.                                                                                                                                                                                                                                                      |
| `pods`     | int     | (Optional) The number of pods for the index to use, including replicas.                                                                                                                                                                                                                                                                                       |
| `replicas` | int     | (Optional) The number of replicas.                                                                                                                                                                                                                                                                                                                            |
| `pod_type` | str     | (Optional) The new pod type for the index. One of `s1`, `p1`, or `p2` appended with `.` and one of `x1`, `x2`, `x4`, or `x8`.                                                                                                                                                                                                                                 |
| `metadata_config` | object | (Optional) Configuration for the behavior of Pinecone's internal metadata index. By default, all metadata is indexed; when metadata_config is present, only specified metadata fields are indexed. To specify metadata fields to index, provide a JSON object of the following form: `{"indexed": ["example_metadata_field"]}` |
| `source_collection` | str | (Optional) The name of the collection to create an index from.                                                                                                                                                                                                                                                                                                |


```js
export const createPineconeIndex = async (client, indexName, vectorDimension) => {
  const existingIndexes = await client.listIndexes();

  if (!existingIndexes.includes(indexName)) {
    const createClient = await client.createIndex({
      createRequest: {
        name: indexName,
        dimension: vectorDimension,
        metric: "cosine",
      },
    });
    await new Promise((resolve) => setTimeout(resolve, 60000));  
  } else {
    console.log(`"${indexName}" already exists.`);
  }
};
```

## 2-updatePinecone.js

![/images/myknowledgetovectordb.png](/images/myknowledgetovectordb.png)


```js
// 1. Import required modules
import { OpenAIEmbeddings } from "langchain/embeddings/openai";
import { RecursiveCharacterTextSplitter } from "langchain/text_splitter";

// 2. Export updatePinecone function
export const updatePinecone = async (client, indexName, docs) => {
  console.log("Retrieving Pinecone index...");

  // 3. Retrieve Pinecone index
  const index = client.Index(indexName);
  // 4. Log the retrieved index name
  console.log(`Pinecone index retrieved: ${indexName}`);

  // 5. Process each document in the docs array
  for (const doc of docs) {
    console.log(`Processing document: ${doc.metadata.source}`);
    const txtPath = doc.metadata.source;
    const text = doc.pageContent;

    // 6. Create RecursiveCharacterTextSplitter instance
    const textSplitter = new RecursiveCharacterTextSplitter({
      chunkSize: 1000,
    });
    console.log("Splitting text into chunks...");
    // 7. Split text into chunks (documents)
    const chunks = await textSplitter.createDocuments([text]);
    console.log(`Text split into ${chunks.length} chunks`);
    console.log(
      `Calling OpenAI's Embedding endpoint documents with ${chunks.length} text chunks ...`
    );

    // 8. Create OpenAI embeddings for documents
    const embeddingsArrays = await new OpenAIEmbeddings().embedDocuments(
      chunks.map((chunk) => chunk.pageContent.replace(/\n/g, " "))
    );
    console.log("Finished embedding documents");
    console.log(
      `Creating ${chunks.length} vectors array with id, values, and metadata...`
    );

    // 9. Create and upsert vectors in batches of 100
    const batchSize = 100;
    let batch = [];
    for (let idx = 0; idx < chunks.length; idx++) {
      const chunk = chunks[idx];
      const vector = {
        id: `${txtPath}_${idx}`,
        values: embeddingsArrays[idx],
        metadata: {
          ...chunk.metadata,
          loc: JSON.stringify(chunk.metadata.loc),
          pageContent: chunk.pageContent,
          txtPath: txtPath,
        },
      };
      batch.push(vector);
      // When batch is full or it's the last item, upsert the vectors
      if (batch.length === batchSize || idx === chunks.length - 1) {
        /*
        await index.upsert({
          upsertRequest: {
            vectors: batch,
          },
        });
        */

        try {
          const upsertResponse = await index.upsert({ 
            upsertRequest: { vectors: batch, }, 
          });  
          console.log(`Upserted ${upsertResponse.upsertCount} vectors`);
        } catch (error) {
          console.log(error);
        }


        // Empty the batch
        batch = [];
      }
    }
    // 10. Log the number of vectors updated
    console.log(`Pinecone index updated with ${chunks.length} vectors`);
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
