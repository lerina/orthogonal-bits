# The Spike Mandala and Rapid Learning Through Question Code

## A Companion to Spike Driven Development

---

## Overview

The Mandala goal-setting method and the rapid learning framework both follow the same shape as Spike Driven Development:

**Start broad. Break things down. Connect the parts. Prioritize. Schedule. Recall. Produce. Iterate.**

But in our way of working, the unit is not a task and not a note. The unit is a **question code** — a spike. A question code is any code written for the purpose of answering a design question. It is disposable by design. Its output is knowledge, not software.

This adaptation rewrites both frameworks so that every step produces question code, harvests essence, and stays non-attached to the scaffolding that produced it.

---

# Part 1: The Spike Mandala

## Goal-Setting as Question Discovery

### Core Idea

Use a 3x3 grid and then a 9x9 grid to move from a vague ambition to concrete, answerable design questions — and from those questions to spikes that you can run, harvest, and delete.

The mandala is not a plan. It is a **map of unknowns**. Its purpose is to help you find the smallest, highest-leverage questions to spike first.

---

## Step 1: Build the 3x3 Baseline

- Draw a 3x3 grid.
- Place your primary goal in the center square. Write it as a verb, not a wish. Not "better system" but "replace the legacy parser."
- Fill the eight surrounding squares with the **high-level design questions** or **key unknowns** that stand between you and the goal.
- Purpose: force yourself to move beyond wishing for a result and identify what you actually do not yet know.

Examples of surrounding squares:

- What does the input actually look like?
- Where does the variation live?
- What is the smallest thing that could work?
- What are the boundaries between parts?
- What data moves between components?
- What constraints does the problem impose?
- What does success look like, concretely?
- What can we delete rather than build?

---

## Step 2: Expand into a 9x9 Grid

- Treat each of the eight surrounding questions as the center of its own 3x3 grid.
- For each question, brainstorm eight **concrete spikes** — specific, small pieces of question code that would answer it.
- Result: up to 64 candidate spikes, each one a concrete experiment.

A good spike is:

- **Concrete:** you can write it in one sitting and run it.
- **Question-shaped:** it answers a specific design unknown.
- **Disposable:** its output is knowledge, not production code.
- **Timeboxed:** 45–90 minutes.
- **Non-attached:** you are willing to delete it after harvest.

Examples of good spikes:

- "Write a 20-line script that parses one real input file."
- "Stub the main flow end-to-end with fake data."
- "Implement the core algorithm for the simplest case."
- "Measure whether the naive approach is fast enough."
- "Try the API with one real request."
- "Sketch the interface and call it from two different places."

Examples of weak spikes:

- "Research the best architecture."
- "Get better at the domain."
- "Refactor everything."
- "Make it production-ready."

When choosing spikes:

- Brain dump first. Do not filter.
- Choose spikes that are distinct from one another.
- Cover multiple unknowns instead of repeating the same experiment under different names.
- Prioritize spikes fully under your control.
- Look for **synergies**: a single spike that answers multiple questions at once is worth ten that answer one each.

---

## Step 3: Classify Each Spike

Before prioritizing, tag each spike as **top-down** or **bottom-up**.

- **Top-down spikes** organize the work. They create the skeleton: main flow, stubs, contracts, boundaries. They answer: *what are the parts, and how do they talk?*
- **Bottom-up spikes** flesh out the skeleton. They implement one concrete case to discover algorithm, data structure, constraints, and edge cases. They answer: *how does this part actually work?*

A healthy project alternates. Top-down spikes create the map. Bottom-up spikes fill in the terrain. Each reshapes the other.

Also tag each spike with the UNIX lens:

- Does it probe a **boundary** between components?
- Does it test a **contract** (input, output, failure modes)?
- Does it explore **plain data** as an interface?
- Does it separate **policy** from **mechanism**?

---

## Step 4: Prioritize and Schedule

- Do not run all 64 spikes at once.
- Identify the highest-leverage questions — the ones whose answers unlock the most others.
- Run only **3–5 spikes in your first week**. A spike is a question, not a task. Fewer, deeper questions beat many shallow ones.
- Add another 3–5 spikes in the second week.
- Continue gradually over 8–12 weeks.
- Schedule spikes as **timeboxed calendar blocks**, not as open-ended to-do items.
- After each spike, write a **harvest note** (see Appendix B of the textbook) and update the mandala.
- Iterate: some spikes will answer their question; others will generate new questions. Redraw the mandala as the terrain becomes clearer.

---

## Why Eight Surrounding Questions?

- A 3x3 grid naturally has eight spaces around the center.
- Eight forces you to cover distinct, meaningful unknowns of your goal.
- It keeps brainstorming manageable and structured.
- It forces prioritization by asking, "What actually makes a difference to what we know?"
- It maps cleanly onto the UNIX philosophy: eight small, sharp, composable concerns rather than one monolithic plan.

---

## Using the Spike Mandala for Work Projects and Deadlines

- Place the project objective in the center, such as "Launch a new product feature."
- Surround it with eight key unknowns or drivers: technical feasibility, market fit, stakeholder constraints, integration boundaries, performance limits, data shape, failure modes, deletion opportunities.
- Expand each into its own eight-part sub-grid of spikes.
- Focus on **controllable, concrete spikes**. For example, choose "Spike the client API with one real request" instead of "Win the client's approval." Choose "Write a 20-line parser for one real file" instead of "Understand the data model."
- Select the highest-leverage spikes, timebox them, harvest them, and delete the scaffolding.
- Meet the deadline not by doing more, but by **knowing earlier**.

---

# Part 2: Rapid Learning Through Question Code

## Core Idea

To learn a subject quickly, build a cognitive framework first. Then attach new information to that structure. Then write **question code** — concrete experiments — to test and deepen your understanding. This mirrors both how experts learn and how Spike Driven Development works.

The unit of learning is not a note. It is a spike.

---

## Step 1: Create a Novice Map

- Set a timer for 30 minutes.
- Write down everything you already know or think you know about the topic.
- Include both broad and niche ideas.
- Purpose: prime your brain and reveal what you know versus what you do not yet understand.
- In our terms: this is your **first harvest**, before any spike. It is the map of your assumptions.

---

## Step 2: Build an Ontology

- Review the contents pages of 4–5 standard textbooks, or the READMEs of 4–5 representative projects.
- Identify 10–20 core categories or sub-topics.
- Purpose: create a syllabus of the field's main components.
- In our terms: this is the **top-down skeleton** of the domain. You are stubbing the parts before you know how they work.

---

## Step 3: Build a Schema

- Write each category on a card or note.
- Place them on a whiteboard or large sheet.
- Draw lines to show how the categories relate.
- Example: "Material enables structure; structure enables form."
- Example: "Parser produces tokens; tokens feed the evaluator; evaluator drives the printer."
- Purpose: create a mental network where new details can attach.
- In our terms: this is the **skeleton map** — the boundaries and contracts between parts, before any part is implemented.

---

## Step 4: Watch Curated Lectures, Then Spike Them

- Use university faculty lectures or high-quality talks.
- Prefer reputable sources.
- Do not take detailed notes during the lecture.
- Immediately afterward, close all materials.
- Spend 15–20 minutes **reconstructing the lecture's key arguments from memory** — this is active recall.
- Then write **question code**: a small, concrete spike that tests one idea from the lecture. Not a summary. Not a diagram. Code that runs.
- Purpose: convert passive understanding into active, executable understanding.

---

## Step 5: Read Academic Papers, Then Spike Them

- Select 3–5 papers.
- Prioritize useful papers that introduce debates or counter-arguments.
- Purpose: understand the field's core tensions and finer details.
- Then write **question code** that tests the tension. If two papers disagree, write a spike that embodies both approaches and compares them concretely.

---

## Step 6: Do Intellectual Fieldwork

- Apply knowledge in the real world.
- Examples: observe buildings for architecture, run thought experiments for philosophy, examine real cases.
- In our terms: **bottom-up spikes against reality.** Run the code on real input. Measure real performance. Break it on real edge cases.
- Purpose: connect abstract ideas to practice.

---

## Step 7: Produce a Tangible Output

- Create an essay, blog post, podcast, audio piece, or project.
- In our terms: **harvest and productionize.** Write the harvest note. If the spike taught you something general, extract the essence into a clean, tested artifact. If it did not, delete the spike and keep the note.
- Purpose: solidify understanding by explaining or building something.

---

## Step 8: Refine Your Novice Map

- At the end of the day, return to your original novice map.
- Use a different colored pen.
- Add new ideas, corrections, and connections.
- Purpose: physically map how your knowledge has progressed.
- In our terms: **update the skeleton map and the mandala.** Some stubs are now real. Some questions are answered. Some new questions have appeared.

---

# Key Learning Concepts

## Active Recall

Active recall means retrieving information without looking at your notes.

Why it matters:

- It strengthens memory.
- It slows the forgetting curve.
- It identifies knowledge gaps.
- After a lecture, it can reduce 24-hour forgetting to about 40%.
- Use it immediately after learning and again throughout the day.

**In our way of working:** active recall is the harvest step. After every spike, close the code. Write down what you learned from memory. Then reopen the code and check. The gap between memory and reality is where the real design lives.

## Question Code

Question code is any code written to answer a design question.

Why it matters:

- It converts vague understanding into concrete, testable knowledge.
- It reveals constraints that no amount of reading can reveal.
- It is disposable by design, so it removes the pressure to get it right.
- It is the smallest unit of both learning and design.

**In our way of working:** every spike is question code. Every question code produces a harvest. Every harvest updates the map.

## Ontology Versus Schema

- **Ontology:** the core categories or components of a subject. It is like a syllabus. It is the top-down skeleton.
- **Schema:** the relationships between those categories. It is like a manual for how the parts connect. It is the skeleton map.

**In our way of working:** the ontology is the list of stubs. The schema is the pipeline between them. Spikes discover both.

---

# Part 3: Integration — The Question Code Loop

Both frameworks and Spike Driven Development converge on the same loop:

1. **Start broad.** Write the ambition or the topic in the center.
2. **Break it down.** Eight surrounding questions or categories.
3. **Expand.** Eight concrete spikes per question.
4. **Classify.** Top-down or bottom-up. Boundary, contract, data, or policy.
5. **Connect.** Draw the skeleton map. Show how the parts relate.
6. **Prioritize.** Choose the few highest-leverage questions.
7. **Schedule.** Timebox the spikes. Put them on the calendar.
8. **Spike.** Write concrete question code. Make it run.
9. **Recall.** Close the code. Reconstruct what you learned from memory.
10. **Harvest.** Write the harvest note. What repeated? What varied? What was accidental?
11. **Decide.** Delete, refactor, or productionize.
12. **Refine.** Update the novice map, the skeleton map, and the mandala.
13. **Repeat.** The project moves forward because understanding compounds, not because scaffolding survives.

---

## Step 4 — Prioritize and Schedule

* Identify high-leverage spikes: Select 1 to 3 spikes that answer multiple questions simultaneously or target your absolute highest-risk unknowns.
* Sequence by risk: Run high-risk, high-uncertainty spikes first to validate core architectural assumptions before committing to a design direction.
* Strict timeboxing: Allocate exactly 45 to 90 minutes per spike. If a spike is unfinished when time expires, stop, document the roadblock, and break it down into a smaller question.

------------------------------
### Execute & Harvest (The Rapid Learning Loop)

[ Formulate Question ] ──► [ Write Disposable Code ] ──► [ Run & Observe ]
                                                                   │
[ Future Production ]  ◄── [ Document Knowledge ]  ◄─── [ Delete/Archive Code ]

## Step 1 — Run the Experiment

* Isolate variables: Write the minimum amount of code required to answer the specific question.
* Ignore production standards: Skip writing tests, optimizing performance, handling unrelated edge cases, or formatting code beautifully.
* Focus on telemetry: Use explicit print statements, log dumps, or debugger inspection to see exactly how data moves and transforms.

## Step 2 — Harvest the Lesson

* Extract the knowledge: Immediately translate code behavior into design principles.
* Document the constraints: Record what failed, what surprised you, and where the hidden boundaries of the problem lie.
* Keep a central log: Save lessons in a simple markdown file (lessons_learned.md) or a team wiki. Never rely solely on memory.

## Step 3 — Clean the Slate

* Discard the artifact: Delete the spike code or move it to a dedicated temporary scratch directory.
* Prevent code rot: Never merge spike code directly into a production branch.
* Separate church and state: Re-write production code from scratch using the mental model you just validated.

------------------------------
### Spike Mandala Reference Template

| Dimension | Top-Down Spike | Bottom-Up Spike |
|---|---|---|
| Primary Goal | Define boundaries, data contracts, and component communication. | Discover core algorithms, data structures, and edge cases. |
| UNIX Lens Focus | Plain data as interfaces; clean policy vs. mechanism separation. | Clear input-output-failure modes; strict boundary validation. |
| Success Criteria | Components integrate seamlessly without knowing internal implementation. | The specific core logic handles extreme inputs and data mutations. |

Now that the framework structure is complete, how would you like to apply it? I can:

   1. Generate a concrete 3×3 or 8×8 example based on a specific project goal you are working on right now.
   2. Create a markdown Spike Tracking Template to log your questions, classifications, and harvested lessons.
   3. Deep dive into the UNIX Lens definitions to clarify how to classify policy vs. mechanism in your code.

Please reply with your choice or share your current engineering goal to begin!

---

The UNIX Lens filters your spikes through the core design philosophy that built the world’s most resilient, composable operating system. Instead of viewing code through modern frameworks or object-oriented hierarchies, the UNIX Lens forces you to think in terms of isolated mechanisms, predictable data streams, and sharp boundaries.
Applying these four definitions to your spikes ensures that your discovered knowledge leads to a radically simple production system.
------------------------------
## 1. Plain Data as Interface

* The Core Concept: Components should communicate using standard, transparent data formats (like JSON, flat maps, plain strings, or byte streams) rather than complex, stateful objects or internal language classes.
* Why it matters: If components share complex objects, they become tightly coupled. If they share plain data, you can swap, rewrite, or test any component in isolation.
* The Spike Question: "Can I decouple these two modules completely by having them pass nothing but a standard dictionary or a flat JSON string?"
* How to run the spike: Write a script where Module A writes its output to a plain text string or memory buffer, and Module B reads it. If you can change Module A’s internal language structure without breaking Module B, your interface is truly decoupled.

## 2. Policy vs. Mechanism

* The Core Concept: Separate Mechanism (the hard functional capability of how something is done) from Policy (the business rules of what should be done and when).
* Why it matters: Mechanisms are durable and stable; policies change constantly based on business whims. Mixing them creates brittle code that is terrifying to refactor.
* The Spike Question: "How do I strip all business logic out of this engine so it becomes a blind, highly efficient executor?"
* How to run the spike: Build a spike that performs a raw technical task—like writing bytes to a disk or making an HTTP request—with zero if/else statements regarding user permissions or business rules. Pass the business rules in as configuration parameters or data inputs.

## 3. Clear Input-Output-Failure Modes

* The Core Concept: Treat every component like a UNIX CLI tool (like ls or grep). It takes a clean input stream (stdin), produces a predictable output stream (stdout), and has explicit, well-defined failure paths (stderr/exit codes).
* Why it matters: Systems don't just fail because of bugs; they fail because they handle unexpected inputs or errors implicitly. Forcing explicit failure modes up front eliminates defensive coding.
* The Spike Question: "What are the exact inputs that cause this logic to crash, and what specific error payloads must it return to protect the rest of the system?"
* How to run the spike: Intentionally feed your spike corrupt data, null values, and massive payloads. Do not try to gracefully handle them; instead, identify the exact boundary where the code breaks and write down the clean error signal it should emit.

## 4. Sharp Boundaries

* The Core Concept: Establish hard lines where context changes. A boundary dictates where data validation happens, where external dependencies are wrapped, and where performance-critical code lives.
* Why it matters: Without sharp boundaries, external API structures leak into your database models, or third-party library updates break your core business domain.
* The Spike Question: "Where exactly does the untrusted outside world end, and where does our clean, trusted application logic begin?"
* How to run the spike: Create a strict "wrapper" or "adapter" script around a third-party SDK or external service. Ensure that no native types from that SDK escape past your wrapper script. If you can replace the SDK with a completely different provider inside the spike within 30 minutes, your boundary is perfectly sharp.

------------------------------
## How to apply the UNIX Lens to your Mandala
When classifying your spikes in Step 3, use these lenses to refine your questions:

* Instead of: "How do I connect to the third-party API?" (Vague)
* Apply the UNIX Lens:
* Boundary: "Can I wrap this API so my app doesn't know it exists?"
   * Contract: "What specific error code does it return when the rate limit hits?"
   * Plain Data: "Can I map its complex nested response into a flat array immediately?"



---

# Final Takeaway

Both frameworks are progressive. So is Spike Driven Development.

Start with a broad goal or topic. Break it into parts. Connect the parts. Choose a few high-impact questions. Write concrete question code to answer them. Recall what you learned from memory. Produce a tangible output. Reflect and iterate.

The mandala is not a plan. It is a map of unknowns.

The rapid learning framework is not a study schedule. It is a spike schedule.

The code is not the product. The code is the question.

The product is the understanding you earn, harvest, and carry forward — while deleting the scaffolding that taught it to you.

> **Existence first. Essence earned. Question code. Spike, harvest, delete, repeat.**

