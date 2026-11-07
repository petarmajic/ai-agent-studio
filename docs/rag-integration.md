# Connecting Agent Studio and RAG Studio

[Overview](../README.md) · [Agent framework](agent-framework.md)

RAG Studio already knew how to search a document collection. My job was to let a chat agent call that search with the collection and retrieval method its user had chosen.

## Keeping the source and method together

A user can give an agent several collections and choose a retrieval method for each one. I worked on the records that store those choices and the mapping that turns them into tools. Each record holds an **agent ID, a collection ID and a retrieval-method ID**.

![RAG Studio prepares documents for search. During chat, Agent Studio uses a collection tool to search the same documents and receive relevant text.](../assets/studio-connection.svg)

When the backend builds a tool, it binds the collection and retrieval method to the wrapper. The model supplies the query at call time; the wrapper already knows where to send it.

## Giving the model enough information to choose

The model needs to choose a source before it sees any retrieved text. I put the collection’s purpose and the names and descriptions of its files in the tool description. With two collections available, the model can use that information to decide which one to search.

| Tool component | Purpose |
| --- | --- |
| Distinct name | Identifies the collection-specific operation in a tool call. |
| Collection and file descriptions | Explain what information the source contains. |
| Retrieval-method description | Explains the search capability available for that source. |
| Query and method inputs | Supply the search request to the wrapper. |
| Bound collection and method | Preserve the source and retrieval choices stored for the agent. |

I bound the collection tools alongside the agent’s selected static tools. The team later added input schemas for the parameters specific to each retrieval method and refactored the service.

## From a tool call to document context

During a chat, the model names a tool and supplies a query. ToolNode runs the wrapper, which passes the query and any method inputs to RAG Studio. The retrieved text comes back as a tool message, so the model can read it before answering or requesting another tool.

The RAG team implemented the searches. I connected them to agents through the following pieces:

| Agent Studio — my integration | RAG Studio — retrieval team |
| --- | --- |
| Store an agent's collection and method choices. | Process documents, create chunks and generate embeddings. |
| Build and describe collection-specific tools. | Implement retrieval methods and their parameters. |
| Bind those tools to the agent model. | Store and retrieve document chunks with PostgreSQL/pgvector. |
| Return retrieval results to the model/tool loop. | Prepare the document context requested by the tool. |

[Agent framework](agent-framework.md) · [Ratings and API tests](testing-and-ratings.md)
