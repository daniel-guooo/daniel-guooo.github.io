---
title: "Multi-Agent Systems Are Really About Designing Parallel Work"
date: 2025-08-06
---

Source: [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system), Anthropic

## A Five-Sentence Summary, by GPT-4o

Anthropic describes how it built a multi-agent research system for open-ended questions that cannot be solved reliably with a fixed pipeline. The lead agent breaks a problem into subquestions, delegates them to subagents, gathers compressed findings, and synthesizes a final answer. The system works best when the task can be split into independent research directions and when the extra token cost is justified by the value of the answer. A large part of the improvement comes from spending more effective reasoning budget through parallel context windows and tool calls, not from some mysterious form of collective intelligence. The harder engineering work is orchestration, evaluation, and reliability: deciding how to split work, avoiding duplicated or misguided searches, judging variable agent paths, and recovering when long-running stateful processes fail.

## What I Think This Article Is Really About

My own takeaway is a little broader: multi-agent systems are not mainly about adding more agents. They are about designing parallel work. The hard part is turning one uncertain task into several well-scoped tasks, running those tasks with the right context and tools, and then merging the results without the coordination cost eating the benefit.

Another way to say this is that a good multi-agent research system is trying to make research low-loss. It has to split the work without losing the question, compress findings without losing the evidence, and merge partial results without losing uncertainty or context.

It is easy to imagine a multi-agent system as one lead agent with a few smaller agents helping in parallel. That picture is not wrong, but it hides most of the hard parts. Once a task is split across agents, the system has to answer questions that do not exist in a simple chat flow:

- Who decides how the problem should be split?
- How do we keep subagents from doing the same work?
- How do we merge partial findings without losing the important context or creating contradictory conclusions?

The article is valuable because it does not sell multi-agent systems as a magical collaboration pattern. It treats them as system engineering. In Anthropic's case, the concrete system is a research system. In my reading, the deeper design problem is parallel task execution: how to split work, assign boundaries, control dependencies, and merge results well enough that parallelism creates more progress than confusion.

That is why I think the most important question is not "How many agents should we use?" It is "What is the right unit of work?"

Research is a good example because many units of work are naturally separable. One agent can investigate a company, another can inspect a time period, another can compare sources, and another can verify citations. But that only works if the boundaries are clear. Every handoff introduces some loss. A lead agent may write an unclear task. A subagent may search with the wrong query. Two subagents may cover the same ground. A good source may be summarized too aggressively. A tool failure may change the future trajectory of the whole run. The architecture only works if the system gets enough upside from parallel exploration to pay for all of that coordination overhead.

### A Simple Framework: Split, Run, Merge

The article gave me a simple way to think about when multi-agent work is actually worth using. I would separate the article's research claim from my broader interpretation like this:

| Layer    | The Article's Point                                                                                             | My Broader Interpretation                                                                                                                                |
| -------- | --------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Split    | Research can be divided into independent directions: sources, hypotheses, entities, timelines, or subquestions. | Multi-agent systems depend on choosing the right unit of work. Bad decomposition creates duplicated effort or hidden dependencies.                       |
| Run      | Subagents can explore in parallel with their own context windows and tool calls.                                | Parallelism is only useful when each task has clear boundaries, the right tools, and enough independence to make progress without constant coordination. |
| Merge    | The lead agent compresses findings into a final answer with citations and synthesis.                            | The merge step is where many systems lose value. Results need evidence, uncertainty, and next steps, not just summaries.                                 |
| Budget   | Anthropic shows that token usage, tool calls, and model choice explain much of the performance variance.        | More parallel work means more cost. The task has to be valuable enough to justify the extra reasoning budget.                                            |
| Recovery | Long-running agents need durable execution, memory, retries, and checkpoints.                                   | Parallel systems fail in ways that can change the whole trajectory, so recovery is part of the architecture, not an add-on.                              |

A hard coding task, for example, may not be a great fit if every worker needs the same live context and every change depends on every other change. A broad research task is different. It may naturally split by source type, time period, company, technical component, or hypothesis. In that case, parallel exploration is not just faster. It can also give the system better coverage.

So the better rule is:

> Multi-agent systems are useful when the work is high-value, information-heavy, parallelizable, and can be decomposed into tasks whose outputs can be merged cleanly.

## Notes I Took From the Article

1. Multi-agent is not just a more complicated version of chat.

The best use case is not every hard task. It is a specific kind of research task: the path is uncertain, the direction changes as new information appears, and the work can be split into relatively independent subproblems. Anthropic's framing is important because research is not a fixed pipeline. It is a process of making a plan, finding something unexpected, revising the plan, and following the next useful thread.

2. Search is not retrieval. Search is compression.

This may be my favorite idea in the article. From the outside, research looks like "finding more pages" or "collecting more documents." But the real value is not the pile of source material. The real value is the compressed insight that comes out of a huge corpus. Multi-agent systems help because different subagents can explore different parts of the information space with their own context windows, then send back the few pieces that matter.

3. Multi-agent systems work partly because they spend more effective tokens.

Anthropic is refreshingly direct about this. In its BrowseComp analysis, three factors explained 95% of performance variance: token usage, number of tool calls, and model choice. Token usage alone explained 80%. That makes the benefit of multi-agent systems less mysterious. They are not automatically smarter because there are more of them. They can be better because they give the system more effective reasoning budget and more room to explore.

4. Multi-agent systems are only worth it for high-value tasks.

The cost side is not small. Anthropic says ordinary agents can use roughly 4x more tokens than chat, while multi-agent systems can use roughly 15x more. That is not something I would want as the default for every user request. The architecture makes sense when the answer is valuable enough to justify the cost: deep research, complex decision support, long information gathering, competitive analysis, due diligence, or other tasks where a better answer is worth real money or time.

5. Not every complex task is a good multi-agent task.

The key question is whether the complexity can be split apart. If all agents need the same shared context at the same time, or if every subtask has strong dependencies on every other subtask, the architecture becomes awkward. Anthropic points out that many coding tasks are not ideal today because the truly parallel surface area is often limited and real-time coordination between agents is still immature. The lesson for me is simple: do not ask whether a task is difficult; ask whether it is parallelizable.

6. The hard part is orchestration, not spawning subagents.

The lead agent has to decide how to decompose the question, how many subagents to use, what each one should investigate, which tools they should use, and what form their outputs should take. If the subagent receives a vague task, it may duplicate work, miss a key source, or run in the wrong direction. This is where a lot of the system's value is created or lost. The orchestrator is not just a manager. It is the part of the system that controls the shape of the information flow.

7. The common failure modes are very concrete.

The article's failure cases are useful because they are not abstract. A system may create dozens of subagents for a simple question. It may keep searching for a source that does not exist. It may start with an overly specific query and accidentally narrow the search space too early. It may use web search when the relevant information lives in Slack. It may give two subagents overlapping assignments and get duplicated work back. These failures are not mainly caused by weak language ability. They are coordination losses.

8. A good research agent starts wide, then narrows.

Anthropic emphasizes a search pattern that feels very close to how a human researcher works. Start with short, broad queries to understand the terrain. Then narrow down once the system knows which entities, sources, terms, or time periods matter. This is more than a search trick. It is a coverage strategy. If the system starts too narrow, the final answer may look confident while being based on a tiny and accidental slice of the information space.

9. Prompting should teach heuristics, not just hard rules.

The prompt engineering lesson I took from the article is that a good agent prompt is not a long list of rigid commands. It should teach research habits: how to decompose a question, how to judge source quality, when to search broadly, when to go deep, when to stop, and how much effort a task deserves. Anthropic even frames good prompts as a collaboration framework. I like that phrasing because a multi-agent prompt is not just telling one model what to do. It is shaping how a whole small organization behaves.

10. Evaluation has to change when the path is not deterministic.

Multi-agent systems may solve the same question through different paths. One run may use three sources, another may use ten, and both may be reasonable. That means evaluation cannot only check whether the agent followed a predefined chain of steps. It has to judge the final result and the health of the process: factual accuracy, citation accuracy, completeness, source quality, tool efficiency, and whether the agent wasted effort or went in circles.

11. LLM-as-judge is useful, but it does not remove the need for human testing.

Anthropic uses LLM judges to evaluate many runs at scale, but the article also makes clear that human review still catches important failures. One example that stood out to me is source quality. A system may prefer SEO-heavy content because it is easy to find, while missing more authoritative PDFs, technical docs, or blog posts. A judge can help scale the evaluation loop, but human testers are still better at noticing the strange failure modes that do not fit neatly into a rubric.

12. Production reliability is about state, not just intelligence.

This section of the article is easy to underestimate, but I think it is one of the most important parts. Agents are stateful, and errors compound. A small failure in a tool call does not just produce one bad step. It can change the next query, the next source, the next summary, and eventually the final answer. That is why a production system needs durable execution, recovery from the point of failure, external memory, regular checkpoints, retry logic, and a way for the agent to adapt when a tool fails.

13. Memory is not just about remembering more.

In a long-running agent system, memory is continuity infrastructure. Anthropic describes storing the research plan in memory because the context can exceed the model's window. It also discusses handoffs when one agent's context gets too full and another agent needs to continue with a cleaner context. That is different from the casual idea of memory as "the model knows more facts." In this setting, memory is what keeps a long task from breaking apart.

14. Synchronous execution is simpler, but it blocks information flow.

Anthropic's lead agent currently waits for subagents to return before moving forward. That makes the system easier to reason about, but it has obvious costs. The lead agent cannot correct a subagent halfway through. Subagents cannot coordinate with one another. A slow subagent can hold up the whole run. This is a classic engineering tradeoff: synchronous orchestration lowers coordination complexity, but it also limits mid-course correction.

15. Asynchronous execution is more powerful, but it makes coordination much harder.

An asynchronous system could keep spawning work, adjust earlier, and let agents make progress at the same time. It would probably raise the ceiling for complex research. But it would also bring harder problems around result coordination, state consistency, and error propagation. Once subagents are moving independently, their local goals can drift, their findings can conflict, and the system needs a stronger way to merge partial states. More parallelism is not automatically better. It has to be matched with a better coordination model.

16. The broader takeaway is parallel task design.

If I reduce the whole article to my own main lesson, it is this: a multi-agent system is not a way to throw more models at a problem. It is a way to design parallel work. Research is Anthropic's best example because research often has natural branches: different sources, different hypotheses, different entities, different time windows, and different verification paths. But the general lesson is about task decomposition. A production system has to decide what can run independently, what must stay centralized, how results should come back, and how to keep the final synthesis from losing the important signal.

## How I Would Apply This To My Own System

The most immediate application for me is my own writing workflow.

If I were building an agent system for my Obsidian notes, I would not start with:

```text
read_entire_vault
write_blog_post
```

That sounds powerful, but it is probably the wrong abstraction. It gives one agent too much context and too much responsibility. The agent has to find the source material, preserve my notes, decide the thesis, invent examples, write the post, and polish the tone in one long run.

I would rather split the work into smaller tasks:

```text
extract_source_claims
extract_my_notes
find_thesis_candidates
design_concrete_example
merge_blog_draft
```

These tasks are not better because there are more of them. They are better because each one has a clear boundary. One task owns the article. One task owns my notes. One task owns the argument. One task owns the example. The lead agent can then merge the outputs instead of carrying the whole process in one context window.

For this to work, each subtask should return something compact and mergeable:

```text
main_findings
evidence_or_source_refs
important_uncertainties
what_not_to_overstate
suggested_next_step
```

That response format matters because the hard part is not launching the subagents. The hard part is making their results easy to combine.

I would also only use this setup when the writing task is large enough. If I am writing a short reaction, one agent is fine. If I am turning a long technical article plus my own raw notes into a publishable post, then parallel work starts to make sense.

That is the main lesson I would take into my own system: more agents are just infrastructure. The real design problem is deciding what should run in parallel, what should stay centralized, and what each task needs to return so the final synthesis still preserves the important thinking.

Thanks for reading :)
