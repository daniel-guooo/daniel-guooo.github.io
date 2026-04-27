---
title: "MCP Is More Efficient When the Model Writes Code"
date: 2025-12-02
---

Source: [Code execution with MCP: Building more efficient agents](https://www.anthropic.com/engineering/code-execution-with-mcp), Anthropic

## A Five-Sentence Summary, by GPT-5

Anthropic argues that direct MCP tool calling becomes inefficient when an agent is connected to hundreds or thousands of tools. Most MCP clients load tool definitions into the model context upfront, and every intermediate tool result also tends to pass back through the model. The article proposes exposing MCP servers as code APIs inside an execution environment, so the agent can discover relevant modules, import only what it needs, and call tools from code. This lets code handle filtering, aggregation, retries, waiting, branching, file persistence, and tool-to-tool data movement before returning a smaller result to the model. The tradeoff is that code execution introduces real operational and security costs, including sandboxing, permission control, resource limits, monitoring, and data-leakage concerns.

## What I Think This Article Is Really About

Code execution with MCP is not just a faster way to call tools. It changes the agent architecture by moving repetitive, stateful, high-volume operations out of the model context and into an executable workspace.

The ordinary tool-calling loop makes the model do too many jobs at once. It has to read the tool catalog, decide which tool to call, receive raw results, inspect intermediate data, carry state in the conversation, copy data from one tool call into another, decide whether to retry, and continue the workflow one turn at a time.

That design works when the tool surface is small and the results are compact. It starts to break down when an MCP server exposes hundreds or thousands of tools, or when a single call returns a large document, a spreadsheet, a Salesforce query result, or a long transcript. The context window becomes an accidental message bus. It holds tool definitions, raw results, temporary artifacts, and workflow state even when the model does not need to reason about most of that material.

Code execution changes the boundary. Deterministic work can move into code: loops, retries, polling, branching, filtering, aggregation, copying data between systems, saving files, and reusing previous workflow logic.

That distinction matters because the model often does not need to know the full content of an intermediate artifact. If the task is to download a meeting transcript from Google Drive and attach it to a Salesforce record, the model may not need to read every word of the transcript. It may only need code that can fetch the transcript, pass it to Salesforce, and report that the operation completed. If the task is to inspect a 10,000-row spreadsheet, the model usually does not need all 10,000 rows in context. It needs the relevant rows, counts, anomalies, or a summary.

This is also why progressive disclosure is so important. Instead of showing the model every MCP tool description upfront, the system can expose a small directory, module index, or search interface. The agent can inspect the available servers, read only the relevant module definitions, import the functions it needs, and keep moving. The interface becomes smaller at the model boundary, but more executable inside the workspace.

The filesystem is the other big shift. If intermediate state can live in files, variables, cached query results, scripts, and workspace artifacts, the agent no longer has to keep everything alive in conversation history. A long task can survive interruptions. A later step can read a saved CSV instead of querying Salesforce again. A useful workflow like "save this Google Sheet as CSV" can become a script, a function, a template, an instruction file, or a skill.

That makes the agent less like a stateless caller and more like an engineering system that can accumulate working methods.

Here is the contrast I would draw:

| Direct Tool Calling | MCP Through Code Execution |
|---|---|
| Tool definitions are loaded into the model context | APIs can be discovered and imported progressively |
| Every tool result tends to flow back through the model | Code can filter, transform, and store intermediate results |
| The model handles loops, retries, and branching one turn at a time | Code handles deterministic control flow directly |
| State mostly lives in conversation history | State can live in files, variables, and workspace artifacts |
| Reuse depends on the model remembering the pattern | Reuse can become scripts, templates, or skills |
| Simpler to operate | Requires sandboxing, permissions, and resource controls |

There is a cost, though. Giving an agent code execution is not a free abstraction. Now the system needs a sandbox. It needs permission boundaries. It needs CPU, memory, and network limits. It needs monitoring. It needs file permissions. It needs protection against malicious code and accidental data leakage.
## Notes I Took From the Article

1. Tool definitions can become their own context-window problem.

    When an MCP client exposes a small number of tools, putting their names, descriptions, parameters, and schemas into context may be fine. The problem changes when the agent is connected to many MCP servers and the combined tool catalog reaches hundreds or thousands of tools. At that point, the model may process a huge amount of tool description text before it even starts working on the user's actual request. The tool catalog itself becomes token-heavy background noise.

2. Intermediate tool results are often more expensive than they are useful.

    Direct tool calling usually sends tool results back into the model context. That sounds natural, but it can be wasteful when the result is large and the model only needs a tiny part of it. A transcript, spreadsheet, document, or CRM query result can consume a large amount of context even if the agent only needs to extract one field, forward the content to another system, or calculate a short summary.

3. The model does not always need to see the data it is moving.

    This is one of the cleanest ideas in the article. If the agent is moving a meeting transcript from Google Drive into Salesforce, the model may not need to read the transcript. It needs to know which source, which destination, and what transformation or validation is required. The actual payload can flow through code, which reduces copying errors, saves context, and avoids making the model inspect data that is not relevant to the reasoning step.

4. Wrapping MCP as a code API enables progressive disclosure.

    Instead of loading every MCP tool definition into the model context upfront, the system can expose a directory or module structure. The agent can first inspect a list of available servers or modules, then load the specific function definitions it needs. That is a better match for how agents already work with filesystems: look around, identify the relevant area, read the necessary files, and ignore the rest.

5. Code is a better substrate for loops, retries, waiting, branching, and polling.

    Many workflows are not a single tool call. They involve waiting for a deployment to finish, polling a Slack channel, retrying a flaky request, branching on a status field, or looping through many records. Making the model take one turn per control-flow step is slow and context-expensive. Code can express the same logic directly, run it in one execution, and return only the meaningful result.

6. Filtering and aggregation should often happen before data returns to the model.

    If a tool call returns 10,000 spreadsheet rows, the model usually should not be the first place where filtering happens. Code can call MCP, filter the rows, compute counts, join records, extract fields, and return only the small set of values the model needs to judge. That changes the model's role from raw-data processor to reviewer of high-signal output.

7. Keeping intermediate state outside the model context improves both efficiency and privacy.

    Intermediate MCP results do not always need to be visible to the model. If raw customer data, contact details, or private documents can stay inside the MCP client or execution environment, the model only sees the logged or returned values. For more sensitive workflows, the client can tokenize or encrypt values and later resolve them through a lookup when another MCP tool needs the real data. The important point is that the model context should not automatically become the place where every sensitive payload is exposed.

8. Filesystem state makes long-running agent work more durable.

    Conversation history is a weak place to store operational state. It gets long, it gets summarized, it may be interrupted, and it is expensive to keep feeding back into the model. If code can write intermediate results to files, the next step can continue from the workspace instead of asking the model to remember everything. A saved Salesforce export, a cached CSV, a progress file, or a generated report can become durable state for the workflow.

9. Reusable scripts and skills turn one-off agent work into accumulated workflow knowledge.

    If an agent writes useful code for "save this Google Sheet as CSV," that code should not disappear after the run. It can become a script, helper function, template, instruction file, or skill. This is the difference between an agent that starts from scratch every time and an agent system that slowly builds a toolbox of working methods.

10. The deeper architecture is model + code execution + MCP APIs + workspace + skills.

    The interesting architecture is not just "LLM calls tool." It is more like this: the model writes code, the code calls MCP APIs, intermediate state lives in the workspace, useful outputs are saved as files, and reusable solutions become skills. That structure gives each layer a clearer job. The model handles intent and judgment. Code handles deterministic execution. MCP connects to external systems. The workspace holds state. Skills preserve successful patterns.

11. Code execution makes agents more powerful, but it also creates a real security and operations surface.

    Running agent-generated code means the system now has to care about sandboxing, permissions, CPU limits, memory limits, network access, filesystem access, monitoring, malicious code, and data leakage. Direct tool calling is more limited, but it is also simpler to operate. Code execution is worth considering when the workflow needs context efficiency, state persistence, and stronger composition, but those benefits come with infrastructure responsibilities.

## How I Would Apply This To My Own System

I would not expose every available MCP tool directly to the model. I would start with a small capability directory: files, git, browser, notes, project metadata, external services, and maybe a few domain-specific modules. The model should be able to inspect what exists, but it should not need to read every full tool schema before it understands the task.

When the agent needs a capability, it can load the relevant API. If it needs note search, load the notes module. If it needs Salesforce, load the Salesforce module. If it needs GitHub, load the GitHub module. The default should be progressive disclosure, not full upfront exposure.

For large results, I would make code do the first pass. Search results, CRM records, spreadsheet rows, logs, browser snapshots, and file lists should not automatically return raw into model context. Code can filter, aggregate, cache, and save them first. The model should receive the final customer context, the top matching notes, the suspicious log lines, the summarized diff, or the next action.

For long tasks, I would write intermediate state into the workspace. If an agent has already queried Salesforce, scanned a repository, extracted a list of TODOs, or generated a draft outline, it should save that artifact. The next step should be able to pick it up from a file instead of recreating the work or relying on conversation history.

For reusable workflows, I would save the working method. A repeated operation like "save Google Sheet as CSV," "extract blog outline from notes," "summarize customer context," or "prepare a release checklist" should become a script, template, instruction file, or skill. That turns repeated agent labor into a local capability.

For sensitive data, I would default to keeping it out of model context. The data can stay in the MCP client, the workspace, or an encrypted lookup. If another tool needs the real value later, code can reference or resolve it through a controlled path. The model should not see raw customer information just because the workflow happened to touch it.

A simple version of the design difference would look like this:

```text
Bad design:

Expose 500 Salesforce tools.
Return 1,000 raw records to the model.
Ask the model to inspect, filter, summarize, and forward them.

Better design:

Expose a small Salesforce module.
Let code query, filter, cache, and summarize records.
Return only the final customer context or next action to the model.
Save reusable query logic as a script or skill.
```

The better design is not just cheaper in tokens. It has a cleaner execution boundary. The model does not need to be the database client, loop controller, scratchpad, serializer, and compliance risk all at once.

I would still treat the execution environment as a serious system component. It needs sandboxing, explicit permissions, CPU and memory limits, network controls, logs, file permissions, and data-leakage protection. The more useful the workspace becomes, the more important those boundaries become.

The point is to give the agent an executable workspace where code handles stateful, repetitive work and the model spends its context on judgment.

Thanks for Reading :)
