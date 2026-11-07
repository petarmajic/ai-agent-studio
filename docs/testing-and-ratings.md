# Ratings and API tests

[Overview](../README.md) · [Agent framework](agent-framework.md)

My work included average user ratings for agents and test cases for the Agent and Project APIs. The team platform also had a separate LLM-based evaluation path for agent conversations.

## Average user rating

Each submitted rating contributes to a stored **sum** and **count**. The displayed average is the sum divided by the count. I changed the model and route behaviour to retain these aggregates and store the average as a floating-point value.

For example, ratings of 5 and 1 produce a sum of 6, a count of 2 and an average of 3.0. This is also the aggregation case exercised by the Agent router tests.

| Stored value | Role |
| --- | --- |
| Rating sum | Retains the total of submitted user scores. |
| Rating count | Retains the number of submitted scores. |
| Average rating | Exposes the aggregate score for the agent. |

This is a user-feedback measure. It does not represent measured task accuracy or retrieval precision.

## Router tests

I added **14 test cases** using the FastAPI TestClient and pytest: seven for agents and seven for projects.

| Agent API | Project API |
| --- | --- |
| Create an agent in a project. | Create a project. |
| List the project's agents. | Reject a duplicate project name. |
| Retrieve an agent by ID. | List projects. |
| Update an agent. | Retrieve a project by ID. |
| Submit scores and check their average. | Handle lookup of a missing project. |
| Retrieve the aggregate rating. | Update a project. |
| Delete an agent and check subsequent lookup. | Delete a project and check subsequent lookup. |

The tests exercise HTTP responses and persisted behaviour across the project and agent lifecycle. They are distinct from the shared RAG evaluation tests and any measurement of model answer quality.

## Shared conversation evaluation

The shared backend extracts tool calls and their results from the conversation trace. Its evaluation path can ask an LLM to score tool choice, tool use and the final answer on a 1–5 rubric, with a heuristic fallback.

That evaluation was a team contribution. It is separate from my user-rating aggregation and Agent/Project test cases. Neither a user-rating average nor an LLM judge score is a benchmark accuracy percentage.

[RAG integration](rag-integration.md) · [Overview](../README.md)
