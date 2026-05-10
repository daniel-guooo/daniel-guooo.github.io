---
title: "Agent Evals Are Living System Tests"
date: 2026-01-09
---

Source: [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents), Anthropic

## A Five-Sentence Summary, by GPT-5.5

Anthropic's article explains why evaluating AI agents is harder than evaluating single-turn chat models or traditional software. Agents use tools, act over many turns, modify external state, and sometimes find valid solutions that static tests were not designed to recognize. A good agent eval needs tasks, trials, graders, transcripts, outcomes, an evaluation harness, and a stable environment. The article separates capability evals from regression evals, and it shows how different agent types need different grading strategies: coding agents, conversational agents, research agents, and computer-use agents all fail in different ways. The main practical lesson is that evals should start early, encode real user expectations, use the right mix of deterministic and model-based graders, and stay healthy as models and products improve.

## What I Think This Article Is Really About

An agent eval has to ask something closer to:

> Did the whole system reach the right outcome, through a process that was reliable, efficient, safe, and repeatable enough for users?

The "whole system" part is important. When we evaluate an agent, we are not only evaluating the base model. We are evaluating the model, the agent harness, the tool interface, the environment, the memory strategy, the retry behavior, the graders, and sometimes even the product assumptions embedded in the task.

That makes agent evals closer to product engineering than benchmark writing.

The eval is where a team turns vague expectations like "the agent should be helpful" into operational definitions:

- What counts as success?
- Should the agent be rewarded for a creative path?
- Which failures are regressions?
- Which failures are actually missing product requirements?

This is why I like the article's emphasis on outcomes and transcripts. The final response is often not enough. A flight-booking agent can say the flight was booked, but the real outcome is whether the reservation exists in the database.

Here is the shift I would draw:

| Simple LLM Eval | Agent Eval |
|---|---|
| Checks one prompt and one response | Checks a multi-step process in an environment |
| Focuses on the final text | Focuses on outcome, trajectory, and side effects |
| Often uses one expected answer | Allows multiple valid paths to success |
| Can be graded with simple matching in some cases | Usually needs multiple graders across different dimensions |
| Mostly evaluates model behavior | Evaluates model + harness + tools + environment |
| Measures correctness once | Measures reliability across multiple trials |

The deeper point is that agent evals force product teams to say what they actually want the system to do. That can be uncomfortable, but it is useful. One prompt change feels better. One tool change seems to help. One new model looks stronger in a few demos. But the team does not know whether the system became more reliable, whether it broke an older workflow, or whether the improvement only worked on the examples people happened to test manually.

Evals turn that vague feeling into a workflow.

They are not perfect. They can overfit. They can miss creative solutions. They can saturate. They can reward the wrong behavior if the grader is too narrow. But without them, an agent product becomes very hard to improve deliberately. The article's most useful framing for me is that evals are a living contract. Early in development, they define what success means. Later, they protect the quality bar. When a stronger model ships, they let a team upgrade faster because the team can test the new model against its own real tasks instead of relying only on public benchmarks or vibes.

That is the part I find most practical: evals are not just measurement after the fact. They are a way to build the product more clearly.

## Notes I Took From the Article

1. Agent evals are harder because agent mistakes compound over time.

    A chat model can be evaluated on a single answer. An agent is different. It calls tools, reads observations, updates state, retries, and continues across many turns. A small mistake early in the trajectory can change the future path of the whole task. That makes the eval problem much closer to evaluating a dynamic system than checking a single output.

2. Sometimes a model finds a better answer than the eval expected.

    The Opus 4.5 example is a good reminder that static evals can be too narrow. The model found a policy loophole in a flight-booking task and technically failed the written eval, even though it produced a better outcome for the user. This is the uncomfortable part of evaluating strong agents: the more capable the system becomes, the more likely it is to find paths the benchmark designer did not anticipate.

3. A task needs defined inputs and success criteria.

    This sounds basic, but it is where many weak evals break. If the task does not say what success means, the grader has to infer it later. That makes the evaluation unstable and makes failures hard to interpret. A good task should be specific enough that two human domain experts would usually agree on the pass/fail verdict.

4. Multiple trials matter because agents are not deterministic.

    One successful run does not mean the agent is reliable. One failed run does not always mean the agent is incapable. The useful question is how often the agent succeeds for the same task under repeated trials. That is why the article treats trials as a first-class part of the eval structure.

5. A grader should score a specific dimension of performance.

    I like the article's advice to avoid one giant judge that decides everything. A task may need separate graders for final outcome, tool use, instruction following, user interaction, source quality, or transcript behavior. Splitting graders by dimension makes failures easier to diagnose and makes the eval less dependent on one vague score.

6. The transcript is part of the evidence.

    The final answer can hide a lot. The transcript shows what the agent actually did: which tools it called, what it observed, which branches it explored, where it wasted effort, and whether it recovered from mistakes. For agent development, transcript review is often where the real debugging happens.

7. The outcome matters more than the agent's claim about the outcome.

    This is one of the cleanest eval lessons. If the agent says "I booked the flight," the grader should check the environment. If the agent says "I fixed the bug," the grader should run the tests or inspect the behavior. Agent evals should grade the state of the world, not just the agent's report about the state of the world.

8. Evaluating an agent means evaluating the model and the harness together.

    An agent is not just a model. The harness controls tool orchestration, memory, prompts, retries, environment access, and result handling. If the eval improves after a harness change, that is a real product improvement even if the base model did not change. If the eval regresses after a prompt change, that is also real. The product is the combined system.

9. Evals are valuable throughout the agent lifecycle.

    Early evals force the team to define success. Later evals protect the system from regressions. After deployment, the same suite can become a quality gate for model upgrades, prompt changes, tool changes, and harness changes. The point is not to create a benchmark once. The point is to keep a useful measurement system alive.

10. Capability evals and regression evals have different jobs.

    Capability evals should start with tasks the agent currently struggles with. They create a hill to climb. Regression evals protect behavior the system already needs to preserve. Over time, a good capability eval can become a regression eval once the system gets good at it.

11. Coding agents need outcome tests, but transcript grading can still help.

    For coding agents, the best first grader is usually deterministic: tests, build checks, type checks, linting, or behavior checks in a stable environment. But once the outcome is covered, transcript grading can add signal. It can catch whether the agent edited unrelated files, skipped important verification, overfit the tests, or used an unnecessarily fragile path.

12. Conversational agents need to be evaluated on both completion and interaction quality.

    For a support or workflow agent, the conversation itself is part of the product. The task may technically complete, but the interaction can still be confusing, rude, too slow, or too dependent on unnecessary clarification. This is where LLM-as-judge can be useful, as long as the rubric is concrete and calibrated against human judgment.

13. Research agents need groundedness, coverage, and source quality.

    Research often does not have one clean ground truth answer. That changes the grading problem. The useful dimensions are whether claims are supported by retrieved sources, whether the answer covers the key facts a strong answer should include, and whether the sources are authoritative enough for the task.

14. Computer-use agents need evals that check both goal completion and interface strategy.

    Agents that operate GUIs have a different tradeoff. DOM-based interactions can be fast but token-heavy. Screenshot-based interactions can be slower but sometimes more token-efficient. A good eval should not only ask whether the agent completed the task. It should also reveal whether the agent chose the right interaction mode for the context.

15. pass@k and pass^k measure different kinds of success.

    pass@k asks whether the agent succeeds at least once across k attempts. That is useful when one successful solution is enough. pass^k asks whether all k attempts succeed. That is more relevant when consistency matters. For many user-facing agents, I care more about pass^k than pass@k, because a user does not want a system that only works after several invisible retries.

16. LLM judges are useful because many agent qualities are hard to specify with traditional metrics.

    Not every quality can be measured with exact matching or unit tests. Tone, helpfulness, groundedness, interaction quality, and recovery behavior often need judgment. LLM judges are useful because they scale approximate human labeling. But they should be used deliberately, with clear rubrics and spot checks, not as magical truth machines.

17. The best time to build evals is early.

    If the team waits until the product is already live, it has to reverse-engineer success criteria from production behavior. That is harder and messier. Early evals are not only for measurement. They help the team decide what the agent is supposed to be.

18. Real user behavior should shape the task set.

    Synthetic tasks are useful, but they should not be the whole suite. The most valuable tasks often come from real user failures, manual testing, support tickets, observed workflows, or production traces. The task set should be prioritized by user impact, not only by what is easy to test.

19. A reference solution proves that the task and graders are sane.

    For each task, it is useful to create a known working solution that passes the graders. This checks that the task is solvable and that the grader is not broken. If strong models fail the task repeatedly, the issue may be the eval design rather than the agent.

20. A balanced problem set prevents distorted behavior.

    If every task requires search, the agent may learn that it should always search. If every task rewards tool use, it may overuse tools. If every task is adversarial, it may become overly cautious. A good suite should include the different situations the real product needs to handle, including cases where the right move is to answer directly, ask for clarification, use a tool, or stop.

21. The eval environment has to be stable and isolated.

    Agent evals are sensitive to environment state. If one trial changes the database for the next trial, or if external services behave differently each run, the score becomes hard to trust. A robust harness should reset state, isolate trials, record traces, and make failures reproducible.

22. Evals should not over-specify the agent's path.

    This is one of the easiest mistakes to make. If the task only rewards one expected sequence of actions, it may punish creative but valid solutions. The better default is to grade the outcome and only add transcript checks when the process itself matters.

23. Partial credit is useful for multi-step agents.

    A binary pass/fail score can hide progress. If the agent completed three of five important steps, that signal matters. Partial grading helps teams see where the agent is improving and where it still fails. It also makes capability evals more informative before the agent can solve the whole task end to end.

24. Eval suites saturate and need maintenance.

    Once agents consistently pass a suite, the suite may still be useful for regression testing, but it stops giving much signal about new capability. That does not mean the eval is bad. It means the suite has changed roles. Teams need to add harder tasks, review grader quality, inspect failures, and keep the suite aligned with the product's next capability boundary.

25. Evals are only one part of understanding agent quality.

    The article is careful about this, and I think it matters. A complete picture needs production monitoring, user feedback, A/B testing, manual transcript review, and systematic human evaluation. Automated evals are powerful because they make iteration faster, but they are not the whole quality system.

## How I Would Apply This To My Own System

The most obvious place I would apply this is my own AI writing and coding workflow.

For example, this blog-writing workflow itself is a good agent eval candidate. The task is not simply:

```text
Write a blog post about an Anthropic article.
```

That is too vague. A better task would define the inputs, the expected outcome, and the grading criteria:

```text
Input:
- Original article link
- My raw notes
- My blog SOP
- Several previous posts from MyBlog

Expected outcome:
- A Markdown draft in the MyBlog directory
- Source link and publish date preserved
- Five-sentence summary
- My own thesis, not only a summary
- A comparison table or clear framework
- Full notes preserved in numbered form
- A final section applying the idea to my own system
- Tone consistent with previous posts
```

Then I would split the graders by dimension.

Some checks can be deterministic:

```text
source_link_present
date_present
required_sections_present
markdown_file_created
notes_are_numbered
code_blocks_are_valid_markdown
```

Some checks need judgment:

```text
thesis_is_specific
style_matches_previous_posts
notes_preserve_the_raw_points
the_application_section_is_concrete
the_post_does_not_sound_like_a_generic_summary
```

And for an agent, I would also grade part of the transcript:

```text
did_read_blog_sop
did_read_previous_posts
did_read_raw_notes
did_check_original_source
did_not_overwrite_unrelated_files
```

That transcript grading is not about forcing one exact path. It is about checking the steps that matter for this workflow. If the agent never reads the SOP, the final draft might still look okay, but the process is not reliable. If it never reads previous posts, it may miss the style. If it never checks the original source, it may amplify mistakes from my raw notes.

For coding tasks, I would use the same structure. The bad eval is:

```text
Did the agent say the bug is fixed?
```

The better eval is:

```text
Did the tests pass?
Did the relevant user flow work?
Did the agent touch only the files needed for the change?
Did it preserve existing behavior?
Did it run verification?
Can the result pass multiple trials, not just once?
```

This is the biggest practical lesson I took from the article: an eval suite is not just a scorecard. It is the product's definition of quality written in executable form.

If the definition is vague, the agent will improve vaguely. If the definition is narrow, the agent may overfit it. But if the eval captures real outcomes, real user expectations, and the important parts of the process, it becomes one of the highest-leverage pieces of the system.

Thanks for Reading :)
