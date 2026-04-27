---
title: "Agent Tools Are Not Just APIs"
date: 2025-11-03
---

Source: [Writing effective tools for agents](https://www.anthropic.com/engineering/writing-tools-for-agents), Anthropic

## A Five-Sentence Summary, by GPT-5

This article argues that designing tools for agents is different from designing APIs for traditional software. An agent is not a deterministic program that calls an interface the same way every time; it explores, misunderstands, retries, and often finds its own path through a task. Because of that, a good tool should not be a thin wrapper around a low-level API, but a task-oriented interface that helps the agent make progress with less confusion. The name, description, parameters, schema, response format, and even error messages of a tool all shape how the agent behaves. The quality of a tool cannot be judged by intuition alone, so it needs to be tested in realistic, multi-step tasks with metrics like accuracy, runtime, tool calls, token usage, and tool errors.

## The Article Is Really About

My main takeaway is this: we used to design APIs for programs, but now we are designing action interfaces for agents.

That changes the problem.

The question is no longer just:

> Can this interface be called?

The better question is:

> Can an agent use this tool to finish a real task reliably, cheaply, and without wandering through too many unnecessary steps?

That is the part I find most important. The article is saying that once the user of the interface becomes a non-deterministic agent, the interface itself needs to be designed differently.

Here is one way to think about the shift:

| Traditional API | Agent Tool |
|---|---|
| Designed for programmers or programs | Designed for LLM understanding and decision-making |
| Low-level, general, composable | Higher-level, semantic, close to a workflow |
| Returns complete data | Returns the data needed for the next action |
| Error codes are written for developers | Error messages should help the agent recover |
| Documentation explains the interface | The tool name, schema, and description are part of the prompt |

For example, a calendar and contacts system might expose tools like this:

```text
list_contacts
get_user
list_events
create_event
```

These make sense to a software engineer. They are clean, simple, and composable.

But for an agent, this design creates a lot of room for waste and mistakes. The agent has to decide which contacts to list, how to filter them, which user fields matter, how to inspect the calendar, how to construct the event, and what to do if any step returns too much or too little information.

A more agent-friendly design might look like this:

```text
search_contacts
schedule_meeting
get_customer_context
search_logs
```

These tools are not better just because they hide more implementation detail. They are better because they move the most repetitive, error-prone, token-heavy parts of the process into deterministic software. The agent can focus on understanding the user's intent and choosing the right action. The tool can handle the structured execution path.

## Notes I Took From the Article

1. In the agent era, a tool is no longer just a thin wrapper around a traditional API.

    It is a contract between a deterministic system and a non-deterministic agent. A normal program calls an interface in a stable, mechanical way. An agent does not. It explores, misunderstands, retries, and may take different paths toward the same goal. That means the tool has to be designed around how an agent thinks and acts.

2. The goal of tool design is not to force the agent into one correct path.

    The goal is to expand the surface area where the agent can successfully act. A good tool system gives the agent more ways to make progress, while still keeping those actions grounded in reliable software.

3. Not every interface that looks reasonable in traditional software is a good interface for an agent.

    Low-level, general-purpose APIs may feel natural to engineers, but they can be awkward for models to use. Interestingly, tools that are ergonomic for agents often end up feeling clearer for humans too.

4. You cannot judge tool quality by intuition alone.

    You have to evaluate tools in real tasks. Strong agent tasks are rarely solved with one tool call. They often require many calls, sometimes dozens, so the evaluation needs to cover multi-step, realistic tool use instead of only checking whether a single call works.

5. Evaluation should not over-specify the agent's path.

    The same task may have several valid solutions. If the test only rewards one expected workflow, you may end up optimizing for an agent that is good at passing the test, not an agent that is good at solving the real problem.

6. Accuracy is not the only metric that matters.

    It is also useful to track runtime, number of tool calls, token usage, and tool errors. These metrics show whether the agent is moving efficiently, going in circles, misusing tools, or wasting context on low-value information.

7. A good tool should save the agent's context instead of exposing all the complexity of the underlying system.

    For example, asking the agent to retrieve every contact and filter them itself is usually worse than giving it a `search_contacts` tool. Asking it to stitch together customer information from several low-level calls is usually worse than giving it a `get_customer_context` tool that returns the useful context directly.

8. The unit of tool design should be the workflow, not the API endpoint.

    Agents are better at acting inside a clear semantic task boundary than navigating a large number of intermediate states and implementation details. A good tool can collapse frequent, multi-step, easy-to-misuse operations into one higher-level action.

9. Tool naming is not cosmetic.

    Prefixes, suffixes, and namespaces can change how an agent understands tool boundaries and chooses between tools. A namespace works like cognitive scaffolding: it helps the model quickly infer which service, resource, or task area a tool belongs to. Anthropic also points out that prefix and suffix choices can behave differently across models, which means naming should be part of evaluation.

10. A tool response should not aim to be as complete as possible.

    It should aim to be as high-signal as possible. The response should help the agent decide what to do next, not mechanically expose every internal field. Values like UUIDs, MIME types, and internal metadata are often useless to the agent and can bury the information that actually matters.

11. Agents are much better with natural-language semantics than with cryptic identifiers.

    Replacing random IDs with meaningful names, descriptions, or even easier-to-reference numbers can improve retrieval accuracy and reduce hallucination.

12. A tool can use a `response_format` option to let the agent choose between `concise` and `detailed` responses.

    That creates a useful tradeoff: the agent can save tokens when it only needs a summary, but still request IDs or other downstream details when it needs to keep working.

13. Error responses need design too.

    A good error response should not only tell the agent that something failed. It should explain what went wrong, what to change, and what a more useful next step might be. Both success responses and error responses shape the agent's behavior.

14. If a tool can return a large amount of content, it should be built from the beginning with ways to fetch less, fetch precisely, and fetch on demand.

    Context windows may keep getting larger, but context is still a limited resource. Token efficiency has to be part of the tool design from day one.

15. Tool descriptions, parameter names, and schemas are prompt engineering.

    They are not just documentation attached to the tool. They are instructions the agent reads while deciding what to do. In many cases, better tool performance may come less from a better model and more from carefully rewriting the tool spec.

## How I Would Apply This To My Own System

The most obvious place I can apply this is my knowledge base.

If I were designing agent tools for my own notes, I would not start with:

```text
list_all_notes
```

That sounds useful, but it would probably be a bad default. It would dump too much information into the context window and force the agent to do the filtering itself.

Instead, I would rather design tools like:

```text
search_notes_by_intent
get_note_summary
find_related_notes
extract_blog_outline_from_notes
```

If I am writing a blog post, the agent does not need to read my entire vault first. It needs the notes that are relevant to the current topic, a short summary of each note, the most useful excerpts, the file paths, and maybe a few related links.

So the default response should probably look more like this:

```text
title
path
summary
relevant_snippets
related_notes
last_updated
```

Then, only when the agent needs more detail, it can call another tool to read the full note.

That is the biggest lesson I took from the article: the goal is not to expose every capability of a system. The goal is to design an action interface that helps an agent understand what it can do, choose the right next step, avoid unnecessary work, and complete the user's task with fewer mistakes.

Thanks for Reading :)
