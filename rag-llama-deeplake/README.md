# RAG LLAMA INDEX 🦙

<p align="center">
    <img src="https://www.peru.travel/contenido/general/imagen/en/430/1.1/portada%20llamas,%20alpacas%20y%20vicu%C3%B1as.jpg" width="500" height="400"/>
</p>

This repository hosts a full Q&A pipeline using llama index framework and Deeplake as vector database. The data used are Harry Potter books that have been extracted from **[Kaggle](https://www.kaggle.com/datasets/hinepo/harry-potter-books-in-pdf-1-7)**. For the following pipeline only 2 books were used due to memory and API KEY tokens limitations.

The main steps taken to build the RAG pipeline can be summarize as follows (a basic RAG Pipeline is performed after text cleaning):

- **Data Ingestion**: import data into the notebook

- **Text Cleaning**: replacing multiple consecutive spaces

- **Nodes**: sentence splitter and windows parser

- **Indexing:** VectorStoreIndex for indexing chunked nodes with associated service and storage contexts

- **Postprocessing**: single sentences are replaced with a window containing the surrounding sentences

- **Reranking**: re-order nodes, and returns the top N nodes

- **Scoring**: top k most similar results

- **Evaluation**: relevancy and faithfulness based on questions generated with the `generate_question_context_pairs` function

Feel free to ⭐ and clone this repo 😉

## 👨‍💻 **Tech Stack**


![Visual Studio Code](https://img.shields.io/badge/Visual%20Studio%20Code-0078d7.svg?style=for-the-badge&logo=visual-studio-code&logoColor=white)
![Jupyter Notebook](https://img.shields.io/badge/jupyter-%23FA0F00.svg?style=for-the-badge&logo=jupyter&logoColor=white)
![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)
![OpenAI](https://img.shields.io/badge/OpenAI-74aa9c?style=for-the-badge&logo=openai&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Git](https://img.shields.io/badge/git-%23F05033.svg?style=for-the-badge&logo=git&logoColor=white)

## 📐 Set Up

In the initial project phase, the documents are loaded and transformed into llama index schema documents. This allows indexing and storing them into a vector store.

Indexing is a fundamental process for storing and organizing data from diverse sources into a vector store (`VectorStoreIndex`), a structure essential for efficient storage and retrieval. This process involves storing text chunks along with their corresponding embedding representations, capturing the semantic meaning of the text (`ServiceContext` incorporates necessary configurations or services needed to generate vector representations). These embeddings facilitate easy retrieval of chunks based on their semantic similarity. Embeddings are typically generated by specialized models like BAAI/bge-small-en-v1.5 (Flag Embedding which is focused on RAG LLMs).

After indexing, a basic query retrieval is performed in order to check the Q&A functioning and performance.

## 🌊 Deeplake RAG Pipeline

### 🪟 Nodes
--------------

When passing documents to a vector store for indexing, there are two main alternatives:

**1. Passing the Whole Document:**

- Involves indexing the entire document as a single unit.
- Suitable for smaller documents that fit comfortably within memory constraints.
- Simpler indexing process but might lack granularity in capturing diverse content within larger documents.

**2. Converting the Document into Nodes:**

- Breaks down the document into smaller, manageable chunks or nodes.
- Ideal for larger documents to prevent memory issues and for better granularity.
- Enables indexing of specific sections or segments, improving the ability to capture diverse content within the document.

As a general guideline, for larger documents, it's advantageous to break them down into smaller chunks or nodes before indexing. This approach not only helps in avoiding memory limitations but also allows for a more detailed and nuanced representation of the document's content. It facilitates better indexing granularity, potentially enhancing the retrieval and analysis of specific sections within the larger document.

This second option is the approach taken for building the Deeplake RAG Pipeline. After creating the nodes the following steps are carried out, which all combined enhance the model performance:

- **Sentence Splitter**: allows to break down your documents into sentences. This will require a pattern to decide where a sentence starts or stop (bullet points, points,..).

- **Sentence Window Node Parser**: this splits a document into nodes, with each node being a sentence. Each node contains a window from the surrounding sentences in the metadata. Additionally, it contains a sentence splitter argument.

These two steps would be enough the create a query and get and answer. However, there are still two additional steps that can further enhance the model performance and are included in the pipeline: postprocessing and reranking.

### 🎖️ Vector Store
--------------
The vector store used to store all embeddings and metadata is **Deeplake**. It is designed for efficient storage and data handling, ensuring maximum efficiency and productivity in LLM applications. All the previously generate nodes, were store and indexed in Deeplake. 

### 🎖️ Postprocessing and Reranking
--------------

- **Post Processing**: during retrieval, before passing the retrieved sentences to the LLM, the single sentences are replaced with a window containing the surrounding sentences using the 
`MetadataReplacementNodePostProcessor`. This is most useful for large documents/indexes, as it helps to retrieve more fine-grained details.


- **Reranking**: after retrieval, the `SentenceTransformerRerank` uses the cross-encoders from the sentence-transformer package to re-order nodes, and returns the `top N nodes`. The crossEncoder model is a type of Sentence Transformer model that takes a pair of sentences and returns a single relevance score.

### 📥 Query Engine
--------------

Once the pipeline is set up, the query can be done. To the question `Which are the main characters of the book?` the following output with the **score** and the top N node from where the information was retrieved.

<p align="center">
<img width="563" alt="Screen Shot 2024-01-05 at 9 05 56 AM" src="https://github.com/benitomartin/benitomartin/assets/116911431/8476121d-b265-4c53-908c-a7c9921baff2">
</p>

The **score** represents the relevance or similarity measure between the query and the retrieved document. This score indicates the likelihood or degree to which the document is considered relevant to the query based on the model's understanding or learned representation of the text.

For instance:

- A higher score typically suggests greater relevance or similarity between the query and the document.
- Lower scores imply lesser relevance or similarity.

### 🚒 Model Evaluation

To evaluate the model the `generate_question_context_pairs` function to generate an evaluation dataset of (question, context) pairs over the text corpus was used. This uses the LLM to auto-generate questions from each context chunk.

After the questions are generated, the model was evaluated in chunk. The metrics used for this purpose were the following:

- **Relevancy** evaluates whether the retrieved context and answer are relevant to the query.

- **Faithfulness** evaluates the integrity of the answer, it faithfully represents the information in the retrieved context (the response from a query engine matches any source nodes) or, in other words, whether there’s a hallucination.

Both metrics evaluate from 0 to 1, the higher the better. The result for the top_k 4 can be seen in the following image.
<p align="center">
<img width="470" alt="Screen Shot 2024-01-05 at 9 05 56 AM" src="https://github.com/benitomartin/benitomartin/assets/116911431/36ad50e5-c3fb-42a2-aaed-53193dbad8eb">
</p>

## 📈 Further Steps

This pipeline shows that a proper text preprocessing combined with a vector database, can end up in a outstanding model performance. However, there is always a path for improvement or different strategies for the data preprocessing. Some steps that can be carried out can be listed as follows:

- Different database: `FAISS`, `Chroma`, `Pinecone`,...

- Data cleaning: removal or numbers, punctuation,...

- Different model: Huggingface `BAAI/bge-reranker-base` reranker model

- Testset Generator: Ragas function `TestsetGeneratorthat` allow to generate questions and answers, which would allow to check metrics like MRR (Mean Reciprocal Rank) or Hit Rate by comparing with the groud truth.
