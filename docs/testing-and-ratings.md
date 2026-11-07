# Ratings and API tests

[Overview](../README.md) · [Agent framework](agent-framework.md)

I worked on agent rating calculations and wrote tests for the Agent and Project APIs. The ratings captured user feedback; the tests checked API responses and changes to stored records.

## Average user rating

I changed the rating model and routes to keep a cumulative sum and count of submitted scores. The average is stored as a floating-point value and updated when a new score arrives.

| User submissions | Rating sum | Rating count | Average |
| --- | --- | --- | --- |
| First score: 5 | 5 | 1 | 5.0 |
| Next score: 1 | 6 | 2 | 3.0 |

This example is also covered by the Agent router tests. Keeping both aggregates means a new submission contributes to the existing feedback rather than replacing the previous score. The resulting average describes user ratings of an agent.

## Router tests

I added **14 test cases** with pytest and FastAPI TestClient: seven for agents and seven for projects.

| Agent API | Project API |
| --- | --- |
| Create an agent in a project. | Create a project. |
| List the project's agents. | Reject a duplicate project name. |
| Retrieve an agent by ID. | List projects. |
| Update an agent. | Retrieve a project by ID. |
| Submit scores and check their average. | Handle lookup of a missing project. |
| Retrieve the aggregate rating. | Update a project. |
| Delete an agent and check subsequent lookup. | Delete a project and check subsequent lookup. |

The cases follow the records through creation, changes and deletion. They also check error responses and the rating calculation, so the coverage includes more than successful create and read requests.

## Conversation evaluation

Other team members developed an LLM-based evaluation path for agent conversations. It reads the tool calls, tool results and final answer, then scores tool choice, tool use and the answer on a 1–5 rubric, with a heuristic fallback.

That evaluation measures a different part of the system from the user-rating average and API tests described here. It reviews conversation behaviour; the router tests check application behaviour and persistence.

[RAG integration](rag-integration.md) · [Overview](../README.md)
