# Agent framework

[Overview](../README.md) · [RAG integration](rag-integration.md)

Agent Studio connects an agent's saved settings to its behaviour in a conversation. I worked on the project and agent APIs, the database models and the integration that builds a LangGraph agent from those settings.

![The web app connects to a shared FastAPI backend for Agent Studio and RAG Studio. The backend uses OpenAI and PostgreSQL with pgvector.](../assets/platform-overview.svg)

## Data model

A project groups agents and document collections. Each agent stores its instructions and selected static tools. A separate configuration record links the agent to a collection and the retrieval method it should use for that collection.

| Record | Main fields | Relationship |
| --- | --- | --- |
| **Project** | Name and description. | Groups agents and document collections. |
| **Agent** | Name, description, selected tools, project reference and rating fields. | Belongs to a project and has collection configurations. |
| **Agent–collection configuration** | Agent ID, collection ID and retrieval-method ID. | Associates a document source with the method selected for that agent. |
| **Collection** | Description, files, chunking and embedding settings. | Provides document context through RAG Studio. |

I implemented CRUD routes, request and response schemas, database operations and configuration routes. I later refactored the agent, project and configuration models and their API logic. Updating an agent also meant updating its collection relationships, so its source selections stayed attached to the correct records.

The backend uses FastAPI and Pydantic for the API layer, with SQLAlchemy and PostgreSQL for persistence. Document processing, chunk storage and vector retrieval are handled by RAG Studio.

## Building the agent

When a chat request arrives, the backend loads the agent's description, selected static tools and collection configurations. The description becomes the system prompt. The selected static tools are combined with the collection-specific RAG tools and bound to the language model.

The graph has two execution nodes: the model and a ToolNode. The model reads the conversation messages and either produces an answer or requests one or more tools. ToolNode executes those calls and adds the results to the message state. Control then returns to the model, which can use the results, make another call or finish its answer.

| Part | Responsibility |
| --- | --- |
| Agent configuration | Determines the instructions, tools and document sources available to the agent. |
| Model node | Reads the messages and produces an answer or tool calls. |
| ToolNode | Executes the requested tools and returns their results to the conversation. |
| Chat API | Supplies request history, runs the graph and returns the answer. |

My changes covered the model/tool integration, instructions for each agent, configured tool selection and chat handling. The chat API receives message history in the request and uses a new thread identifier for each call.

## Tool interface

The common tool interface provides a name, description, execution method and LangChain adapter. I worked on this interface, the weather integration and the API that lists available static tools. Other team members added Jira, SMTP email and public-holiday tools.

Only the tools selected for an agent are exposed to its model. Collection retrieval uses the same calling interface, with wrappers built from the agent's document-source configuration. This lets the execution graph handle both external operations and document search through ToolNode.

## API responsibilities

| API area | Behaviour |
| --- | --- |
| Projects | Create, list, retrieve, update and delete projects. |
| Agents | Manage agent records, selected tools and collection configurations. |
| Tools | List static tool definitions available for agent configuration. |
| Chat | Load an agent, run its graph and return the response. |
| Ratings | Accept user scores and return the accumulated average. |

[Ratings and API tests](testing-and-ratings.md) · [Overview](../README.md)
