# Ratings and API tests

[Overview](../README.md) · [Agent framework](agent-framework.md)

Users could rate an agent after a chat. I changed how the backend accumulated their scores and wrote tests for the Agent and Project APIs.

## Average user rating

Each new score adds to the agent’s rating sum and increases its rating count by one. The route divides the updated sum by the count and stores the average as a floating-point value.

| User submissions | Rating sum | Rating count | Average |
| --- | --- | --- | --- |
| First score: 5 | 5 | 1 | 5.0 |
| Next score: 1 | 6 | 2 | 3.0 |

The rating test submits those two scores in order and checks that the second response contains 3.0. A separate test reads the rating back through the API.

## Router tests

I wrote **14 router test cases** using pytest and FastAPI’s TestClient: seven for agents and seven for projects.

| Agent API | Project API |
| --- | --- |
| Create an agent in a project. | Create a project. |
| List the project's agents. | Reject a duplicate project name. |
| Retrieve an agent by ID. | List projects. |
| Update an agent. | Retrieve a project by ID. |
| Submit scores and check their average. | Handle lookup of a missing project. |
| Retrieve the aggregate rating. | Update a project. |
| Delete an agent and check subsequent lookup. | Delete a project and check subsequent lookup. |

The deletion tests make two requests: first they delete the record, then they try to fetch it by ID. The second request must return 404.

## Conversation evaluation

Other team members built a separate evaluation for conversations. It reads the tool calls, their results and the final answer. An LLM scores the conversation on a 1–5 rubric for tool choice, tool use and answer quality. The evaluator also has a heuristic fallback.

A user rating records what someone thought of an agent. The conversation evaluator scores a particular exchange, and the router tests check that the API returns the expected responses.

[RAG integration](rag-integration.md) · [Overview](../README.md)
