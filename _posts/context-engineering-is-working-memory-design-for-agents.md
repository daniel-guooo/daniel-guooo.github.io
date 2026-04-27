---
title: "Context Engineering Is Working Memory Design for Agents"
date: 2025-10-27
---

Source: [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents), Anthropic

## A Five-Sentence Summary, by GPT-5

Anthropic's article argues that context engineering is becoming a core discipline for building useful agents, because an agent has to decide what information, tools, and history to keep available while it works. The article treats context as more than the prompt: it includes system instructions, tool definitions, MCP servers, retrieved files, message history, intermediate observations, and any other tokens the model can use. A larger context window does not remove the problem, because model performance can degrade when the context becomes long, noisy, or poorly organized. The article walks through practical patterns such as clear system prompts, examples, agentic search, hybrid context loading, compaction, structured note-taking, and sub-agent architectures. Its main engineering lesson is that agent builders need mechanisms for deciding what enters the context, when it enters, what gets compressed, what gets stored outside the context, and what gets delegated elsewhere.

## What I Think This Article Is Really About

My main takeaway is this: context engineering is not prompt engineering with a larger context window. It is the engineering discipline of managing an agent's working memory over time.

That is not just a change in terminology. The engineering object has changed.

In the one-shot query era, the main question was usually how to write the prompt. How should the system prompt be phrased? Which examples should be included? How should the instruction be ordered so the model gives a better answer in one call?

That still matters, but it is no longer enough once the task is handed to an agent. An agent is not a single model call. It is a process that loops through reasoning, tool use, observations, partial results, mistakes, recoveries, and updated plans. As it works, it keeps producing more state. At every step, the system has to answer a harder question: what should the next context window contain?

That is why I think the real shift is from expression quality to working-memory management.

Prompt engineering mostly cares about the quality of one interaction. Context engineering cares about the memory state of a long-running agent process. The problem is not simply "can the model see more?" The more useful question is: what should the model see, when should it see it, and what should remain after it has seen it?

Here is the cleanest way I can frame the difference:

| Prompt Engineering | Context Engineering |
|---|---|
| Optimizes one interaction | Manages a long-running agent process |
| Focuses on wording and examples | Focuses on working memory, tools, history, and retrieval |
| Assumes the key information is already present | Decides what information should enter the context and when |
| Treats context as input | Treats context as a limited resource with diminishing returns |
| Improves the next answer | Improves continuity across many steps |

The important point is that context is not free storage. A larger context window can be useful, but it does not automatically make the agent smarter. If the context is noisy, stale, redundant, or too large, the agent can still lose track of the task. The model may technically be able to read the tokens, but that does not mean it can reliably use the right tokens at the right moment.

I find it useful to think of context engineering as five operations:

- Load: put stable, high-value instructions and constraints into the context up front.
- Search: let the agent discover relevant information just in time instead of dumping everything in at the beginning.
- Compress: turn old conversation history and tool results into a smaller, higher-signal task state.
- Remember: store important progress, constraints, and decisions outside the current context window.
- Delegate: send detailed exploration to subagents so the main agent's memory does not get filled with every low-level detail.

That framework is also why this article feels bigger than prompt design. It is really about the life cycle of information inside an agent system.

## Notes I Took From the Article

1. Prompt engineering for one-shot queries is no longer enough for agent workloads.

    Earlier prompt engineering was mostly designed for a world where the user asked a question, the model answered, and the interaction ended. Agents create a different workload. They need to carry data, tool results, message history, file references, MCP tools, and task state across many steps. The engineering problem becomes deciding which pieces of information should be handed to the next context window, not just how to phrase the current instruction.

2. Context has diminishing returns as it gets larger.

    More context does not automatically mean better performance. As the context grows, the model can become worse at accurately extracting the information it needs. Anthropic describes this as context rot. The way I read it, the context window should be treated as a limited resource with diminishing marginal returns. Every extra token may help, but it may also dilute the model's attention away from the high-signal parts of the task.

3. Long context is hard partly because models have more experience with shorter sequences.

    The model's attention patterns are learned from its training distribution, and shorter sequences are usually more common in that distribution. That means the model has less experience handling dependencies across very long contexts. Even though a transformer can theoretically connect tokens across a sequence, the practical burden grows as the sequence gets longer. The model still has ability, but precision retrieval and long-distance reasoning become easier to break.

4. A system prompt should be clear, direct, and written at the right altitude.

    The best system prompt is not a huge rulebook, and it is not a vague slogan either. It should explain concepts in simple language that an agent can actually use while making decisions. The right altitude is specific enough to guide behavior, but flexible enough to give the model strong heuristics instead of forcing it into brittle instructions.

5. Examples are the "pictures" worth a thousand words.

    This point stood out to me because it explains why examples often work better than long abstract instructions. A good example gives the model a concrete pattern to imitate. It shows what the desired behavior looks like in context. For agents, examples can be especially useful because they demonstrate not only the final answer, but also the style of tool use, decomposition, and recovery that the system expects.

6. A simple definition of agents is: LLMs autonomously using tools in a loop.

    I like this definition because it keeps the concept grounded. An agent does not need to be mysterious. The key difference is that the model can take actions, observe results, and continue the loop without the user manually controlling every step. Once that loop exists, context becomes a moving target. The agent is constantly creating new information that may or may not deserve to stay in memory.

7. Agentic search is different from giving the model all the information up front.

    The point of agentic search is not to stuff every document, file, or database result into the prompt. It is to let the model discover useful information as it works. This self-managed context window keeps the agent focused on relevant subsets instead of drowning it in exhaustive but potentially irrelevant material. In practice, that means the system should often provide tools and lightweight references first, then let the agent load the real content only when it has a reason.

8. Hybrid context loading is usually more practical than pure upfront context or pure autonomous search.

    Some information is stable and low-change, so it makes sense to put it into the context early. A `Claude.md`-style file is a good example: repo conventions, project rules, and persistent instructions can be useful from the beginning. Other information is too dynamic or too broad, so the agent should search for it at runtime. The real design choice is the autonomy boundary: what should humans curate ahead of time, and what should the model be trusted to discover on its own? As models improve, I expect systems to move toward letting intelligent models act more intelligently, with less manual curation.

9. Compaction is mainly about preserving continuity.

    When a long conversation approaches the context limit, the agent can lose earlier constraints, forget important decisions, or become inconsistent with its previous work. Compaction exists to prevent that. Its purpose is not to write a pretty summary. Its purpose is to extract the important state from the old context so the model can continue as if it has moved to a fresh scratchpad while still carrying the real memory of the task.

10. Good compaction is a high-fidelity summary of task state.

    The hard part of compaction is choosing what to keep. My takeaway is that compaction should first optimize for recall, then gradually improve precision. In other words: it is better to keep a little too much than to drop a detail that later turns out to be important. After the important state has been preserved, the system can remove redundancy and low-value information. The most valuable thing to preserve is verified working memory: decisions, constraints, completed work, open risks, and facts that have already been checked.

11. Structured note-taking creates external memory for long-running work.

    Instead of relying only on the current context window, an agent can actively write important information into an external store. That could be a to-do list, a project note, a `Claude.md` file, or some other durable workspace. This is especially useful when a task has many milestones or when progress needs to survive context resets. Without external memory, an agent can take many steps but still fail to accumulate stable knowledge across those steps.

12. Sub-agent architectures protect the main agent's working memory.

    A subagent may read a large amount of information, explore a messy branch of the task, or inspect details that might not end up mattering. The main agent should not have to carry all of that raw context. A better architecture lets the subagent do the detailed work in its own clean context window, then return only a small, distilled result to the main agent. This keeps noisy exploration from filling the main agent's memory and lets the main agent focus on synthesis, prioritization, and final decisions.

13. Compaction, structured note-taking, and sub-agent architectures solve different long-context problems.

    I would not treat these as interchangeable techniques. Compaction is strongest when the task requires a lot of back-and-forth and the system needs to preserve conversational flow. Structured note-taking is strongest for iterative development with clear milestones, where the agent needs an external place to record durable progress. Multi-agent architectures are strongest for complex research and analysis, where parallel exploration is worth the coordination cost and where only the final distilled results should return to the main agent.

## How I Would Apply This To My Own System

The first place I would apply this is my AI coding workflow.

The bad default is to ask one agent to read a large codebase, keep every search result, remember every instruction, track every open question, implement the change, and review its own work in one long context. That may work for small tasks, but it does not scale well. The agent's context fills up with raw tool output, old hypotheses, irrelevant file contents, and stale branches of reasoning.

For a coding task, I would rather make the context policy explicit:

```text
Load: repo instructions, the user request, known constraints, and the smallest relevant entry points
Search: use targeted file search and code search to discover the real implementation surface
Compress: keep a verified task-state summary instead of every raw tool result
Remember: write durable milestones, decisions, and open risks into an external note when the task is long
Delegate: send bounded research or review work to subagents and bring back only the distilled findings
```

For example, if I am asking an AI coding agent to refactor part of a codebase, I do not want the agent to load the whole repository just because the context window can fit it. I want it to start with the task, the repo-level instructions, the failing test or relevant feature path, and a few likely files. Then it should use search to discover adjacent code, tests, call sites, and conventions.

After that, the useful state is not "everything the agent has seen." The useful state is smaller and more structured:

```text
files_that_matter
current_assumptions
confirmed_constraints
implementation_plan
tests_run
test_results
open_risks
decisions_made
```

That is the information I would want preserved during compaction. Old raw grep output, full stack traces that have already been diagnosed, and abandoned hypotheses usually do not need to remain in the main context forever. They can be summarized or dropped once their value has been extracted.

I would also use structured note-taking more deliberately. For long projects, the agent should not rely on chat history as the only memory. It should write a short task note that records what has been changed, what still needs verification, which files are sensitive, and what assumptions have already been checked. That note becomes a stable bridge across context resets.

Subagents are useful when exploration is valuable but noisy. A main coding agent might keep the global plan, while a subagent inspects a legacy module, another reviews test coverage, and another checks whether a migration pattern has precedent elsewhere in the repo. Each subagent can read a lot, but the main agent should only receive the conclusion, evidence, and risks. That is the whole point: exploration can be large, but the returned context should be small.

The practical lesson for me is that an AI coding workflow should not be measured by how much context it can carry. It should be measured by how deliberately it manages context while it works. The goal is to keep the agent's working memory focused, verified, and useful over time.

Thanks for reading :)
