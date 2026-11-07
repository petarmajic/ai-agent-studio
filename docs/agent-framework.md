# Agent framework

[Overview](../README.md) · [RAG integration](rag-integration.md)

Two agents in the same project can have different instructions, tools and document sources. I built the APIs that store those choices and worked on the LangGraph runtime that uses them when a chat starts.

![The web app connects to a shared FastAPI backend for Agent Studio and RAG Studio. The backend uses OpenAI and PostgreSQL with pgvector.](../assets/platform-overview.svg)

## Data model

An agent belongs to one project. Its record holds the agent description and the names of the static tools the user selected. Document search needs a separate record because the same agent can use a different retrieval method for each collection.

| Record | Main fields | Relationship |
| --- | --- | --- |
| **Project** | Name and description. | Groups agents and document collections. |
| **Agent** | Name, description, selected tools, project reference and rating fields. | Belongs to a project and has collection configurations. |
| **Agent–collection configuration** | Agent ID, collection ID and retrieval-method ID. | Associates a document source with the method selected for that agent. |
| **Collection** | Description, files, chunking and embedding settings. | Provides document context through RAG Studio. |

I wrote the project and agent routes, their request and response schemas, and the database operations behind them. I later refactored the models and configuration routes. One detail mattered when editing an agent: if a user changed the search method for an existing collection, the update had to change that binding; if they removed the collection, it had to remove the binding too.

FastAPI handles the requests, Pydantic defines their fields and SQLAlchemy maps the records to PostgreSQL. RAG Studio handles document processing, chunk storage and vector search.

## Building the agent

For each chat request, the backend reads the agent’s description and uses it as the system prompt, with a default prompt if the description is empty. It looks up the selected static tools by name, builds a search tool for each configured collection and binds the available tools to the model.

The LangGraph loop has two nodes. The model reads the messages and produces either an answer or one or more tool calls. ToolNode runs the requested tools and adds their results to the conversation. The model reads those results on its next turn.

| Part | Responsibility |
| --- | --- |
| Agent configuration | Determines the instructions, tools and document sources available to the agent. |
| Model node | Reads the messages and produces an answer or tool calls. |
| ToolNode | Executes the requested tools and returns their results to the conversation. |
| Chat API | Supplies request history, runs the graph and returns the answer. |

I also worked on chat handling. The client sends the conversation history with each request, and the API assigns a new thread ID for that call.

## Tool interface

I built `GET /tools` so the configuration screen could list each available tool by ID, name and description. When a chat starts, the backend looks up the tools selected for that agent and binds their LangChain adapters to the model.

I implemented the weather tool. If a user asks “What’s the weather in Zagreb?”, the model can call it with the city name. The tool looks up the first matching place in Open-Meteo, then requests current conditions using its coordinates and time zone. It returns the temperature in °C and wind speed in km/h. If the place lookup fails, the tool returns a message explaining that it could not resolve the location.

The team used the same interface for three other tools.

| Tool | What the agent can do |
| --- | --- |
| Jira Cloud | Create an issue in the configured project. On success, Jira returns an issue key and the tool builds a link to the ticket. |
| Email | Send a message to specified recipients through a configured SMTP server, including Gmail SMTP. The tool returns the recipient list and subject after sending. |
| Croatian public holidays | Look up holidays or long weekends through Nager.Date. If there are no upcoming holidays left this year, the upcoming-holidays lookup checks the following year. |

## API responsibilities

| API area | Behaviour |
| --- | --- |
| Projects | Create, list, retrieve, update and delete projects. |
| Agents | Manage agent records, selected tools and collection configurations. |
| Tools | List static tool definitions available for agent configuration. |
| Chat | Load an agent, run its graph and return the response. |
| Ratings | Accept user scores and return the accumulated average. |

[Ratings and API tests](testing-and-ratings.md) · [Overview](../README.md)
