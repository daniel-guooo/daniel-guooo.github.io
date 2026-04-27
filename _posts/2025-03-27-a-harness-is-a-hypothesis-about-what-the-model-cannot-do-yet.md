---
title: "A Harness Is a Hypothesis About What the Model Cannot Do Yet"
date: 2026-03-27
---

Source: [Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps), Anthropic

## A Five-Sentence Summary, by GPT-5.4

Anthropic describes a harness for pushing Claude beyond naive long-running coding by combining task decomposition, explicit evaluation, and model-specific context management. The post starts from two recurring failure modes: agents lose coherence as context grows, and they are too generous when grading their own work. It first tests a generator/evaluator loop on frontend design, using rubrics for design quality, originality, craft, and functionality plus Playwright MCP evaluation. It then scales the pattern to full-stack app building with a planner, generator, evaluator, sprint contracts, and QA that exercises the running application. The most important engineering lesson is that harness components should be treated as temporary scaffolding: useful when they compensate for current model weaknesses, and candidates for removal when a stronger model no longer needs them.

## What I Think This Article Is Really About

What I think this article is really about is not a three-agent architecture. The planner/generator/evaluator setup is the concrete implementation, but it is not the deeper idea. The deeper idea is harness design.

My main takeaway is this: harness design is about building temporary scaffolding around the current model's weaknesses, then constantly stress-testing which parts are still load-bearing as models improve.

That framing matters because it keeps the architecture from turning into a cargo cult. A planner is not valuable because "good agent systems have planners." A planner is valuable if the current model under-scopes the product, makes weak early framing decisions, or starts coding before it has a coherent spec. An evaluator is not valuable because "multi-agent is better." It is valuable if the current model cannot reliably judge its own output. Context reset is not valuable because "long tasks need handoff." It is valuable if the current model loses coherence or develops context anxiety as the conversation gets long.

Every harness component is really an embedded assumption about the model's current capability boundary:

- If I add a planner, I am assuming the model does not consistently turn a short prompt into a strong product and technical direction on its own.
- If I add an evaluator, I am assuming the model's self-evaluation is too optimistic or too shallow.
- If I add a rubric, I am assuming the model needs steering toward the qualities I care about, not just measurement after the fact.

Those assumptions can be true, but they can also go stale. The article's Opus 4.5 to Opus 4.6 transition is the important signal here. Some scaffolding that mattered for one model became less necessary for the next one. The task did not become easy, and harness engineering did not disappear. The useful harness simply moved: less effort spent maintaining coherence through sprint decomposition, more effort spent catching deeper product gaps, richer interactions, and last-mile feature completeness.

 I read it as a much more useful engineering stance: identify the failure mode, add the smallest scaffold that changes the outcome, then revisit that scaffold when the model changes.

The better the model gets, the less obvious the scaffolding becomes. But the harness problem does not go away. Stronger models let us attempt longer, messier, more open-ended work. The frontier moves from "can it keep a coding task coherent for two hours?" to "can it make a browser DAW whose core interactions are actually usable?" to "can it build a product that has taste, depth, correctness, and a working critical path?" The harness space does not shrink. It shifts toward the next capability boundary.

## Notes I Took From the Article

1. Long-running agents fail mainly because of coherence decay and weak self-evaluation.

    The article starts from two problems that show up again and again in long-running application development. First, as the context window fills, the model can lose the thread of what it is building. Second, when the model is asked to judge its own work, it often grades too generously. These are different problems: one is about continuity over time, and the other is about judgment. A good harness may need to address both, but it should not pretend they are the same thing.

2. Context reset and compaction solve different continuity problems.

    Compaction shortens the history so the same agent can continue with a compressed version of the conversation. Context reset starts a fresh agent and relies on a structured handoff to carry over the state of the work. That difference matters because some models have context anxiety: as they approach what they think is the context limit, they start wrapping up too early. In that case, compaction may preserve continuity but not remove the anxiety. A reset gives the next agent a clean slate, though it also adds orchestration cost and makes the handoff artifact much more important.

3. Harness choices depend heavily on the model version.

    This is one of the most useful details in the post. In earlier work, Sonnet 4.5 had enough context anxiety that context reset was essential. In the Opus 4.5 harness, that behavior was less severe, so the build could run as one continuous session with automatic compaction. With Opus 4.6, the model improved enough that Anthropic could remove the sprint construct and still run a long build coherently. That progression makes the main lesson concrete: a harness is not an eternal architecture. It is a model-dependent scaffold.

4. Self-evaluation is unreliable because generators tend to overrate their own work.

    Frontend design is a good test case because it exposes self-evaluation failures quickly. A page can be functional and still be bland, generic, visually timid, or obviously AI-generated. When the same model that produced the page is asked to judge it, it tends to praise the output too confidently. That is not only a design problem. Coding agents can do the same thing with product quality, edge cases, and "looks done" implementations that have not been properly exercised.

5. Separating generator and evaluator turns generation and judgment into two separately optimizable roles.

    The point of an evaluator is not simply to add another agent. The point is to stop asking one agent to be both builder and critic at the same time. It is easier to tune a standalone evaluator to be skeptical than to make a generator harshly critical of its own work while it is also trying to build. Once the evaluator becomes a separate role, its prompt, rubric, tools, and failure modes can be improved independently from the generator's. That separation gives the generator a concrete external target to iterate against. In the article, this separation becomes practical because the evaluator is not just reading code or judging screenshots in the abstract. It uses Playwright MCP to operate the page, take screenshots, observe interactions, and check whether the running app behaves the way the sprint contract says it should. That tool access makes the evaluator closer to a QA agent than a text-only reviewer.

6. Rubrics are steering mechanisms.

    The frontend experiment used four criteria: design quality, originality, craft, and functionality. The interesting move was weighting design quality and originality more heavily, because Claude was already relatively strong on basic craft and functional correctness. That changed the model's behavior. It pushed the generator toward more aesthetic risk instead of safe, default-looking UI. Even the wording mattered: a phrase like "museum quality" nudged the output toward a particular visual style. So a rubric is not neutral measurement. It is part of the prompt surface that shapes the output space.

7. Evaluator feedback does not only fix bugs; it can raise the generator's ambition.

    The generator/evaluator loop did not merely make outputs cleaner. Across iterations, the generator often reached for more ambitious solutions in response to evaluator critique. In frontend design, that meant less template-like work and more distinctive visual direction. The article's museum example is a good illustration: after several iterations, the model moved from a polished but expected landing page to a spatial gallery experience. That kind of jump is the part I find most interesting. The evaluator was not just pulling the generator away from mistakes; it was expanding what the generator tried to do.

8. Planner is valuable because early product and technical framing errors propagate through the whole build.

    The planner's job is not to micromanage implementation. Its value is upstream: product scope, product context, high-level technical shape, and ambition. If the planner writes a bad low-level implementation plan too early, that mistake can cascade through the rest of the build. But if there is no planner at all, the generator may under-scope the app, start coding too quickly, and produce something narrower than the user intended. The planner is useful when the task needs stronger framing before code exists.

9. Sprint contracts align generator, evaluator, and user intent before implementation starts.

    The sprint contract is a small but important mechanism. The product spec is intentionally high-level, so the generator still needs to decide what a particular sprint will actually build. Before coding, the generator proposes what "done" means and how success should be verified. The evaluator reviews that contract until both sides agree. That gives the build a testable target before implementation begins, and it reduces the chance that the generator writes code for a version of the task that the evaluator or user did not actually want.

10. The full harness moved the result from "looks like it works" toward "the critical path actually works."

    The 2D retro game maker comparison makes the value of the harness visible. The solo run was cheaper and faster, and at first glance it looked close to the prompt: there was a level editor, sprite editor, entity behavior system, and play mode. But the core game loop was broken. Entities appeared, but the game could not really be played. The full harness was far more expensive, but it produced a broader spec, a more polished interface, richer tools, AI-assisted generation, and most importantly, a play mode where the core interaction actually worked. That is the difference between UI that resembles a product and a product whose critical path is real.

11. Even a strong harness still exposes product intuition gaps and edge cases.

    The full harness did not make the retro game maker perfect. The workflow still did not clearly teach the user that sprites and entities should be created before filling a level. Physics had rough edges. Some generated level content created awkward or blocked play. The article treats these as useful signals, not as reasons to declare the harness failed. A harness can lift the model past a major failure mode while still revealing the next one. In this case, the next target might be product intuition, onboarding flow, edge-case exploration, or deeper interaction testing.

12. Every harness component is a hypothesis about what the model cannot do yet, so it should be re-examined when the model improves.

    This is the sentence I would put at the center of the whole post. The Opus 4.6 update shows the right maintenance behavior: remove one scaffold at a time, check whether performance degrades, and keep only what is still load-bearing. The updated browser DAW run is a good example. Opus 4.6 could handle a long build without sprint decomposition, but the evaluator still caught real gaps: clips that could not be dragged, recording that was still a stub, and effect editors that were just numeric sliders instead of graphical interfaces. Those are not cosmetic bugs. They are core interactions for a DAW. The old coherence scaffold became less important, but verification around feature completeness still mattered.

## How I Would Apply This To My Own System

For a small project, I would begin with the simplest loop that can plausibly work: one strong coding agent, a clear prompt, a few deterministic checks, and maybe Playwright MCP if the result is a web app. Then I would look at the trace and ask what actually failed.

If the failure is unclear scope, I would add a planner. The planner's job would be to turn a rough idea into a product spec, define the core user flows, and make high-level technical choices without freezing every implementation detail too early.

If the failure is overconfident self-evaluation, I would add an evaluator. For frontend work, I would probably start with the same four dimensions from the article: design quality, originality, craft, and functionality. I would not use the rubric only as a score sheet. I would treat the wording as steering. If I want more visual risk, I should say so in the criteria. If I want the output to avoid generic SaaS UI, the evaluator and generator should both see that preference.

If the failure is long-task drift, I would add continuity scaffolding: compaction, structured handoff, context reset, or smaller work chunks. But I would choose between them based on the model's behavior. If the model simply needs less history, compaction may be enough. If it starts prematurely wrapping up because the conversation feels long, reset plus handoff may be better.

If the failure is implementation drift, I would add a sprint contract. Before the agent writes code, it should state what it is about to build, what "done" means, and how the result will be verified. That contract gives the evaluator something concrete to test and gives the generator a tighter target.

If a new model can reliably handle one of these problems on its own, I would remove that harness piece and test again. A harness should be removable, testable, and replaceable. It should not become permanent architecture just because it worked once.

Thanks for Reading :)
