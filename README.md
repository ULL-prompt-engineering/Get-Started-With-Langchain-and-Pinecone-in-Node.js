- [Get Started With Langchain and Pinecone in Node.js](#get-started-with-langchain-and-pinecone-in-nodejs)
  - [Back to the tutorial](#back-to-the-tutorial)
  - [References](#references)
  - [Versions](#versions)
  - [File Loaders](#file-loaders)
  - [Understanding indexes](#understanding-indexes)
  - [Understanding collections](#understanding-collections)
  - [Computing the vector dimension](#computing-the-vector-dimension)
    - [Number of vectors](#number-of-vectors)
    - [Dimensionality of vectors](#dimensionality-of-vectors)
  - [Execution](#execution)
  - [Pinecone Indexes Dashboard](#pinecone-indexes-dashboard)

# Get Started With Langchain and Pinecone in Node.js

## Back to the tutorial

You are in lesson [Pinecone](https://github.com/ULL-prompt-engineering/prompt-engineering-101/tree/main#lesson-7-pinecone) of the [Prompt Engineering 101](https://github.com/ULL-prompt-engineering/prompt-engineering-101/blob/main/README.md) course.

## References

* See Developers Digest youtube tutorial: https://youtu.be/CF5buEVrYwo. This crash course will guide us through the basics of programming with Langchain and Pinecone.
* [Prompt Engineering 101](https://github.com/ULL-prompt-engineering/prompt-engineering-101/blob/main/README.md) course

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
* [LangChain File Loaders](https://js.langchain.com/docs/modules/data_connection/document_loaders/integrations/file_loaders/)

## Understanding indexes

Standalone vector indices like [FAISS](https://www.pinecone.io/learn/faiss/) (**Facebook AI Similarity Search**) can significantly improve search and retrieval of vector embeddings, but they lack capabilities that exist in any database. 

Vector databases, on the other hand, are purpose-built to manage vector embeddings, providing several advantages over using standalone vector indices:

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


An index is the highest-level organizational unit of vector data in Pinecone. 
It accepts and stores vectors, serves queries over the vectors it contains, and does other vector operations over its contents. 


Each index runs on at least one **pod**[^1].

[^1]: **seed pod**. noun. a **carpel** (female reproductive organ) or **pistil** (usually is located in the center of the flower is made up of three parts: the stigma, style, and ovary) enclosing the seeds of a plant, esp a flowering plant. So **pineconce pods** are the seeds of the pinecone.

**Pods** are pre-configured units of hardware for running a Pinecone service. 

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
