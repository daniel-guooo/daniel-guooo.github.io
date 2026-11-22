---
title: "Agent Memory Is a Prediction, Not a Record"
date: 2026-11-06
---

Source: [The components of agent memory](https://www.anthropic.com/research/components-of-agent-memory), Anthropic

## A Five-Sentence Summary, by Claude Opus 5.5

Anthropic's article looks at agent memory as knowledge an agent gains from experience and carries across sessions as readable context. It describes memory as a stack of layers: raw transcripts at the bottom, notes the agent chooses to write, representations such as linked wikis built on top of those notes, and a small amount of injected context loaded into every session. The central problem is that an agent has to decide what is worth remembering before it knows what future sessions will need, and models differ a lot in how well they make that call. The article's answer is "dreaming," an offline process inspired by sleep consolidation in the human brain, where the agent reviews its transcripts and notes to fix mistakes, fill gaps, build wikis, and decide what deserves to be loaded up front. It closes with open questions about when memory should be written, how to keep it current, and how much structure is worth maintaining on top of plain transcript search.

## What I Think This Article Is Really About

My main takeaway is this: memory is not a record of what happened. It is a prediction about what will matter.

I used to think about memory, both an agent's and my own, mostly as storage. You save things, and the hard part comes later, when you need to find the right thing at the right time. After reading this article, I think the harder part comes earlier. When an agent writes a note, it is betting that this piece of information will be useful in a future session it cannot see yet. The note is a forecast, not a log entry.

Once I started reading memory that way, a few things in the article fell into place for me.

If memory is a prediction, then it can be wrong. That means it needs feedback. A note that is never checked against reality is just a guess that keeps sitting there. This is the part I kept coming back to while reading, because it felt a lot like an agent loop: act, get feedback, correct. The difference is that the thing being corrected here is not the answer to a task. It is the memory itself.

If memory is a prediction, then it needs a target. You cannot decide what is worth remembering without knowing what the memory is for. This is also why I think soft guidance works better than rigid rules for strong models. Telling a model why memory matters gives it something to predict against. Telling it exactly what to record replaces its judgment with yours.

And if memory is a prediction, more is not automatically better. Writing everything down is not a good prediction. It is avoiding the prediction.

Here is the shift as I see it:

| Memory as a Record | Memory as a Prediction |
|---|---|
| Saves what happened | Bets on what will matter later |
| Better memory means more coverage | Better memory means better bets |
| Written once and kept | Checked, revised, merged, or deleted |
| The hard part is retrieval | The hard part is deciding what to keep and checking it later |
| Organized by when it was written | Organized by what it is for |
| Grows by default | Has to be compressed to stay useful |

The article's answer to "how do you check the predictions" is dreaming: an offline pass that goes back through the raw transcripts, compares notes against what actually happened, fixes what is wrong, and reorganizes what has grown too large. I like this answer, but it does not fully settle the question for me. For an agent, the transcript can serve as ground truth. For other kinds of memory, it is less obvious what the feedback should be, how often to look for it, and when a note should be deleted rather than updated.

## Notes I Took From the Article

1. Agent memory has layers: transcripts, notes, representations, and injected context.

    Transcripts are the raw record of what the agent did. Notes are what the agent decides to write down. Representations, like a wiki, are built on top of those notes and capture how the pieces relate to each other. Injected context is the small set of notes that every session loads automatically. I find it useful to think of these as layers, because each one moves further from the raw record and closer to what the agent can use right away.

2. Memory has to be predicted, and models differ in how well they predict.

    Every note is a guess about what a future session will need, and different models are not equally good at making that guess. Weaker models tend to remember everything. Stronger models try to predict what will actually matter. The main problems I see are two: writing everything down, and writing things down without ever verifying them.

3. Strong models do better with soft guidance than with hard rules.

    For strong models, it works better to gently steer memory behavior than to prescribe it. Hard rules hurt. Specific human instructions about how to write memory can actually make the model's memory worse.

4. Memory should be updated and corrected continuously.

    Memory is not something you write once and trust forever. Notes can be wrong from the start, or become wrong as the environment changes. My own view is that correction needs a basis: some kind of feedback from the environment that tells you whether to change a note, how to change it, and when.

5. Building representations like a wiki matters.

    A wiki helps the model manage a growing body of memory. It saves tokens, gives the memory structure, and lets the model load what it needs progressively instead of all at once. The condition is that the wiki has to actually compress and distill. Otherwise it is just another copy of the source.

6. Wikis mostly help complex tasks that need multi-step retrieval.

    When a task requires pulling information from many places and connecting it, a wiki is useful. When the task is simple and the answer can be found directly, plain search is usually better.

7. Injected context should be handled carefully.

    The benefit of loading memory into every session is that the agent does not need to search for it. The cost is that it consumes tokens, carries any mistakes into every session, and longer instruction files are harder for the model to follow. The better approach is to keep injected context small: keep an index at the top, push the details down to lower layers, and leave links so Claude can follow them when it needs to. Claude Code, for example, only loads the first 200 lines or 25KB of its MEMORY.md index at startup.

8. A good approach is to store memory around a goal.

    Decide what the memory is for first. Then let that purpose decide how the memory is organized and what counts as important.

9. Dreaming is the main thread of the article.

    Dreaming is an offline process that goes back over transcripts and notes. It works across every layer: it fixes mistakes, writes new notes, builds wikis, and decides what belongs in injected context. The article compares it to the human brain, where the hippocampus records experiences quickly and sleep later consolidates them into long-term memory. The basis for correction can be the transcript itself. In one Pokémon run, Claude wrote down the wrong location of a landmark in Mt. Moon, and dreaming caught it by comparing the note against the transcript and fixing it.

10. Dreaming can fill gaps across sessions.

    A single session only sees its own context. Dreaming can look across sessions, repair broken links, and add pages that are missing.

11. One open question is when memory should be written.

    The unsolved problem is how to create memory that captures what later tasks will need. Writing during the task captures what matters in the moment, but it pulls attention away from the main task. Writing afterward through dreaming does not interfere with the task, but the new memory is not available right away.

12. Another open question is how to keep memory current.

    The focus should be on information that is likely to change, and on continually adjusting how often memory gets updated.

13. Search plus representations works well, but it has a cost.

    Using transcript search as the foundation and adding representations like a wiki on top tends to work better than either alone. But maintaining those representations costs compute, and errors or hallucinations can build up over time.

Thanks for Reading :)
