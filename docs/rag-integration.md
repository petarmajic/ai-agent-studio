# Connecting Agent Studio and RAG Studio

[Overview](../README.md) · [Agent framework](agent-framework.md)

RAG Studio provides document collections and retrieval methods. Agent Studio needs to expose the selected combinations as tools a model can call during a conversation. I worked on the stored configuration, the agent integration and the tool wrappers connecting these parts.

## What the binding represents

An agent's collection configuration links three things: **the agent, a collection and the retrieval method selected for that collection**. The same agent can therefore use several configured document sources. Each source keeps its own retrieval-method choice.

![Agent configuration becomes separate retrieval tools, which dispatch to RAG Studio and return context to the model](../assets/rag-tool-bridge.svg)

The configured collection and retrieval method stay associated from tool construction through retrieval dispatch.

## Building a callable retrieval tool

| Step | Information carried forward | Purpose |
| --- | --- | --- |
| Read agent configuration | Collection reference and selected retrieval-method identifier. | Preserve the user's source and method choices. |
| Load collection context | Collection name, description and available document descriptions. | Tell the model what information the tool can retrieve. |
| Construct the wrapper | A distinct tool name, retrieval callable and description. | Expose collection retrieval through the agent's tool interface. |
| Bind the tool set | Configured static tools plus collection-specific RAG tools. | Make document retrieval available during agent execution. |
| Dispatch a call | Query, selected collection and method inputs. | Delegate retrieval to RAG Studio. |
| Return context | Retrieved text associated with the requested collection. | Give the model context for its next response or tool decision. |

I added the agent-side collection-to-method mapping and integrated it with the configured tool set. I also worked on the wrappers and their descriptions, including collection and file information. The shared implementation later added method-specific parameter schemas and further service refactoring.

## Why the tool description matters

An agent may have more than one source available. The wrapper's description identifies the collection, explains the retrieval method and includes document information. This gives the model information for choosing a tool before retrieval has happened.

The wrapper carries the selected collection into the retrieval call. It exposes that collection's retrieval capability without requiring the agent runtime to implement the retrieval algorithm itself.

## Responsibility boundary

| Agent Studio integration | Shared RAG Studio components |
| --- | --- |
| Agent and collection configuration. | Document upload, chunking and embedding workflows. |
| Selecting the configured collection/method combinations. | Retrieval-method implementations and their parameters. |
| Constructing and exposing retrieval tools to the model. | Chunk storage and vector retrieval with PostgreSQL/pgvector. |
| Returning retrieved context into the agent's execution loop. | Retrieving and preparing the requested document context. |

My contribution was the connection between the studios and the agent execution path. The platform's retrieval algorithms were developed by the RAG Studio team.

[Agent framework](agent-framework.md) · [Testing and ratings](testing-and-ratings.md)
