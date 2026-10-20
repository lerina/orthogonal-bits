# Spike Driven Development: A Language-Agnostic Textbook

**Subtitle:** Existence First, Essence Earned  
**Motto:** Spike, harvest, delete, repeat.

---

## Preface

This is a textbook about how to design software when you do not yet know what you are designing.

It is language-agnostic. It does not teach a syntax, a framework, or a paradigm. It teaches a mindset and a loop:

1. Write concrete code to discover what the design actually is.
2. Use the UNIX philosophy to keep parts small, sharp, and composable.
3. Use **top-down spikes** to organize the work and create a skeleton.
4. Use **bottom-up spikes** to flesh out that skeleton.
5. Stay non-attached to disposable code. Keep the lesson; delete or refactor the scaffold.
6. Let the iterative process drive the project forward.

The central philosophical claim is simple: **existence precedes essence**. In software, this means the abstraction does not exist first in a perfect Platonic heaven. It emerges from working, concrete examples. You discover the generic by writing the specific. You discover the architecture by building the skeleton. You discover the essence by spiking the details.

This textbook is for individuals, pairs, and teams who want to replace upfront paralysis with iterative discovery.

---

# Part I: Philosophical Foundations

## Chapter 1: Existence Precedes Essence

### Learning Objectives

- Understand why upfront architecture often fails.
- Learn to treat concrete code as the source of design knowledge.
- Recognize the psychological barriers that spikes remove.

### 1.1 The Platonic Trap

Many developers begin with an imagined ideal system: perfect interfaces, perfect layers, perfect abstractions. This is the Platonic trap. It assumes the essence of the system can be known before any concrete instance exists.

But software design is not geometry. You do not sit down and write generic, “Platonic” algorithms cold. You discover them by looking at what your code actually does. The necessary abstractions and constraints on types emerge naturally from concrete implementations.

### 1.2 Discovery over Assumption

Do not assume the abstraction. Discover it.

Write the most concrete version first. Hardcode values. Duplicate logic. Avoid generics. Make it compile and run. Then write a second concrete case. Feel the friction. Only then ask: what repeated? What varied? What was accidental? What did the compiler or runtime force?

### 1.3 Grounded Learning

Humans learn through examples adjacent to their experience, not through abstract concepts alone. Writing concrete iterations lets you identify patterns in your own code. Functional patterns, design patterns, and architectural patterns become easier to understand because they are linked to familiar problems rather than theoretical text.

### 1.4 Subordination of Details

Through repeated iteration, you learn to subordinate unnecessary detail. By writing the initial code, you can identify which parts are specific to a single use case and which parts can be safely abstracted. This is how you build up functionality through layering and composition.

### 1.5 Overcoming Paralysis

For junior developers especially, the pressure to “get it right” via upfront architecture is paralyzing. Writing concrete code removes this barrier. You have thousands of bad programs inside you. The only way to get to the better, more abstract design is to type them out and compile them.

### 1.6 The Motto

> Existence first. Essence earned. Spike, harvest, delete, repeat.

---

## Chapter 2: The Spike

### Learning Objectives

- Define a spike and distinguish it from a prototype.
- Learn to formulate a design question.
- Use timeboxes and spike cards.

### 2.1 What Is a Spike?

A **spike** is a timeboxed, concrete experiment whose purpose is to answer a design question. It is not production code. It is not a prototype you intend to keep. It is a learning instrument.

A spike can be written in any language, any style, any paradigm. Its only requirement is that it runs and teaches you something.

### 2.2 The Design Question

Every spike begins with a question. Not “build feature X,” but:

- What does this need to know?
- Where does the variation live?
- What is the smallest thing that could work?
- What are the boundaries between parts?
- What data moves between components?
- What constraints does the problem impose?

A spike without a question is just procrastination with syntax highlighting.

### 2.3 Timeboxing

Timebox the spike. 45–90 minutes is typical. The timebox forces you to focus on the question, not on polishing the code. When the timebox ends, you harvest the lesson and decide what to do next.

### 2.4 Types of Spikes

- **Top-down spike:** Organizes the work. Starts at the highest level and stubs lower layers. Discovers interfaces, flow, and boundaries.
- **Bottom-up spike:** Fleshes out the skeleton. Implements one concrete piece to discover algorithms, data structures, constraints, and edge cases.
- **Research spike:** Explores an unknown technology, library, or protocol.
- **Performance spike:** Measures whether an approach is fast enough.
- **Interface spike:** Tests whether a proposed interface is usable and sufficient.

### 2.5 The Spike Card

Keep a short card for each spike:

```text
Question:
Concrete case 1:
Concrete case 2:
Friction / duplication:
Invariant:
Variation:
Candidate abstraction:
What the compiler forced:
Delete or keep:
```

This turns spikes into a design journal.

### 2.6 Spike vs Prototype vs Production

- A **spike** is disposable. Its output is knowledge.
- A **prototype** is a draft you may evolve.
- **Production code** is tested, documented, and maintained.

Never confuse them. Fossilized spikes become legacy nightmares.

---

## Chapter 3: Non-Attachment and Disposable Code

### Learning Objectives

- Treat code as scaffolding for understanding.
- Practice deleting or refactoring spike code.
- Use version control to manage disposable work.

### 3.1 Code as Scaffolding

Spike code is sacrificial. Its job is to teach. Once the lesson is learned, the code has served its purpose. Keeping it around out of sentimental attachment slows the project.

### 3.2 Keep the Lesson, Delete the Code

The project moves forward because understanding compounds, not because prototype code survives. Write down the lesson in a harvest note. Then delete the spike branch or refactor the useful parts into production.

### 3.3 Version Control and Branch Hygiene

Use throwaway branches for spikes. Name them clearly: `spike/parse-unknown-format`. Do not merge them directly. Instead, create a clean branch and implement the discovered design with tests.

### 3.4 Emotional Discipline

Non-attachment is a skill. You are not your code. The spike is not a child. It is a question mark made of instructions. Delete it with gratitude.

---

# Part II: UNIX Philosophy as Design Compass

## Chapter 4: Small, Sharp, Composable

### Learning Objectives

- Apply UNIX philosophy to language-agnostic design.
- Use clear interfaces and plain data.
- Separate policy from mechanism.

### 4.1 Do One Thing Well

Write components that do one thing and do it well. A component should have a single reason to change. This makes it easier to spike, test, replace, and compose.

### 4.2 Composition and Clear Interfaces

Build programs to work together. Composition requires clear contracts: what goes in, what comes out, what errors can occur. A good interface is small, explicit, and stable.

### 4.3 Plain Data as Universal Interface

Prefer plain, simple data structures for communication between components. In UNIX, text streams are the universal interface. In language-agnostic terms, use the simplest data representation that preserves meaning: records, lists, maps, tagged unions. Avoid passing complex objects across boundaries when plain data will do.

### 4.4 Separation of Policy and Mechanism

Separate decisions from execution. A policy component decides what should happen. A mechanism component performs the action. This separation makes both easier to spike independently.

### 4.5 Silence, Repair, and Experimentation

- **Silence is golden:** Components should not produce unnecessary output.
- **Repair:** When something breaks, fix the smallest part.
- **Prototype before polishing:** Use spikes to explore before committing to production.

### 4.6 UNIX Checklist

- Is this component small enough to understand in one sitting?
- Does it do one thing well?
- Can it be composed with other components?
- Is the interface plain and explicit?
- Is policy separated from mechanism?
- Can I spike this part independently?

---

## Chapter 5: Top-Down Spikes: Organizing the Work

### Learning Objectives

- Use top-down spikes to create a skeleton.
- Discover boundaries and contracts.
- Organize parallel work.

### 5.1 Skeleton First

A top-down spike starts at the highest level. You write the main flow as if the lower-level pieces already exist. You stub them. You define their names, inputs, and outputs. The result is a skeleton: a working structure with empty or fake parts.

Example pseudocode:

```text
main:
  input = readInput()
  parsed = parse(input)
  result = process(parsed)
  output = format(result)
  writeOutput(output)
```

`parse`, `process`, `format`, and `writeOutput` do not exist yet. They are stubs. But the main flow exists. The architecture is visible.

### 5.2 Writing the Main Flow with Stubs

The stubs should be honest. They should return plausible data or fail loudly. They should not contain hidden logic. Their purpose is to let the skeleton run end-to-end.

### 5.3 Discovering Boundaries and Contracts

By writing the main flow, you discover:

- What data moves between steps.
- What each step needs to know.
- Where the variation lives.
- Which parts are independent.

These discoveries become contracts. A contract is a promise: given these inputs, return these outputs, or fail in these ways.

### 5.4 Parallelism and Team Organization

A good skeleton enables parallel work. Once contracts are clear, different people can spike different stubs. The UNIX philosophy supports this: small tools with clear interfaces can be built independently and composed later.

### 5.5 UNIX Pipelines as Metaphor

Think of your system as a pipeline:

```text
input | parse | process | format | output
```

Each stage does one thing. Each stage communicates via plain data. You can spike any stage in isolation. You can replace any stage without rewriting the whole pipeline.

### 5.6 Skeleton Map

Keep a skeleton map:

```text
[readInput] -> [parse] -> [process] -> [format] -> [writeOutput]
     |            |           |           |            |
   stubbed      stubbed     stubbed     stubbed      stubbed
```

As bottom-up spikes replace stubs, update the map.

---

## Chapter 6: Bottom-Up Spikes: Fleshing Out the Skeleton

### Learning Objectives

- Use concrete cases to discover algorithms and constraints.
- Apply the rule of three.
- Replace stubs with real implementations.

### 6.1 Concrete Cases

A bottom-up spike takes one stub and implements it for one concrete case. No generics. No abstractions. Just make it work for a single input.

Then write a second concrete case. Copy-paste deliberately. Mutate it. Feel the friction.

### 6.2 Rule of Three

Two is coincidence. Three is pattern. Do not abstract after one example. After two, you may see a hint. After three, you have enough evidence to extract an essence.

### 6.3 Discovering Constraints and Types

As you write concrete cases, constraints emerge. You discover that `parse` needs to handle only integers, or that `process` must be pure, or that `format` needs a locale. These constraints become part of the interface.

### 6.4 Performance, Edge Cases, and Algorithms

Bottom-up spikes are ideal for exploring:

- Which algorithm is fast enough?
- What happens with empty input?
- What happens with malformed input?
- What are the memory limits?
- What are the error cases?

### 6.5 Replacing Stubs

When a bottom-up spike has taught you enough, replace the stub with a real implementation. If the implementation is still experimental, keep it behind the same contract. The skeleton remains stable; the details evolve.

### 6.6 When to Stop

Stop when the question is answered. Stop when the timebox ends. Stop when the next spike would teach you less than the cost of writing it.

---

# Part III: The Iterative Loop

## Chapter 7: Spike, Harvest, Refactor, Repeat

### Learning Objectives

- Run the core iterative loop.
- Alternate top-down and bottom-up spikes.
- Harvest essence and decide what to keep.

### 7.1 The Cycle

1. **Ask a design question.**
2. **Spike concretely.** Top-down to organize, bottom-up to flesh out.
3. **Observe what happens.** Run it. Break it. Measure it.
4. **Harvest the pattern.** What repeated? What varied? What was accidental?
5. **Decide:** delete, refactor, or productionize.
6. **Repeat.**

### 7.2 Alternating Top-Down and Bottom-Up

Top-down spikes organize the work. They create the skeleton, the contracts, the boundaries.

Bottom-up spikes flesh out the skeleton. They discover the details, the constraints, the algorithms.

Alternate between them. Let the skeleton guide the bottom-up work. Let the bottom-up discoveries reshape the skeleton.

### 7.3 Harvesting Essence

The harvest is the most important step. Ask:

- What is the invariant?
- What is the variation?
- What did the compiler or runtime force?
- What did I wish I had named?
- What is the smallest abstraction that makes all concrete cases simpler?

If the abstraction does not make all cases simpler, it is wrong.

### 7.4 Deciding: Delete, Refactor, or Productionize

- **Delete:** The spike answered the question; the code is not needed.
- **Refactor:** The spike contains useful logic; extract it into production code with tests.
- **Productionize:** The spike is already close to production; add tests, documentation, and error handling.

### 7.5 Metrics of Learning

Track:

- Number of design questions answered.
- Number of spikes per question.
- Time from question to harvest.
- Number of abstractions discovered vs invented.
- Amount of spike code deleted vs refactored.

These metrics measure learning, not lines of code.

---

## Chapter 8: Testing and Preservation

### Learning Objectives

- Use TDD to preserve discovered design.
- Write characterization and contract tests.
- Productionize spikes safely.

### 8.1 TDD Preserves Discovered Design

TDD says: tests drive design. Spike Driven Development says: spikes discover the design; tests preserve it.

Spike first when the shape is unknown. Then write tests around the essence you found. Then productionize.

### 8.2 Characterization Tests

Characterization tests capture what the spike actually does. They are useful when you are refactoring a spike into production and need to ensure behavior is preserved.

### 8.3 Contract Tests

Contract tests verify that a component honors its interface. They are especially important when multiple components are composed, as in a UNIX-style pipeline.

### 8.4 Productionizing a Spike

1. Harvest the lesson.
2. Write a clean implementation from scratch or refactor the spike.
3. Add tests for the contract and edge cases.
4. Delete the spike branch.
5. Update the skeleton map.

---

## Chapter 9: Refactoring and Abstraction

### Learning Objectives

- Subordinate unnecessary detail.
- Build through layering and composition.
- Avoid premature DRY.

### 9.1 Subordinate Unnecessary Detail

After several spikes, you can see which details are specific to one use case and which are structural. That is where layering and composition begin.

### 9.2 Layering and Composition

Build layers: a core layer of pure logic, a boundary layer for input/output, a policy layer for decisions. Compose small components into larger ones.

### 9.3 Naming Variation

Most bad abstractions capture what is the same and ignore what changes. The variation is the design. Name the variation explicitly.

### 9.4 Avoiding Premature DRY

Do not remove duplication before you understand the variation. Duplication is evidence. It shows you what is repeated and what is different. Only after the rule of three should you abstract.

### 9.5 When Abstraction Is Wrong

An abstraction is wrong if:

- It has only one caller.
- It makes simple things complicated.
- It hides important variation.
- It requires constant exceptions.
- It was invented, not discovered.

---

# Part IV: Practice

## Chapter 10: Team-Level Spike Driven Development

### Learning Objectives

- Integrate spikes into team workflow.
- Use harvest notes and architecture decisions.
- Manage disposable code at scale.

### 10.1 Spike Branches and Demos

Normalize spike branches and throwaway code. Demo spikes, not slides. Show what you learned, not what you imagined.

### 10.2 Harvest Notes and Architecture Decisions

Record architecture decisions as “what we discovered from spikes,” not “what we imagined upfront.” A harvest note should include the question, the concrete cases, the pattern, and the decision.

### 10.3 Pairing and Mobbing

Pair a driver who writes the concrete case with a navigator who watches for patterns. Mob a top-down spike to align on boundaries. Mob a bottom-up spike to explore a tricky algorithm.

### 10.4 Onboarding and Knowledge Sharing

New team members can run spikes to learn the system. They discover the design by writing concrete code, just as the original team did.

### 10.5 Managing Disposable Code

- Keep spikes on separate branches.
- Never merge spike code directly.
- Delete spike branches after harvest.
- Keep harvest notes in the repository.

---

## Chapter 11: Anti-Patterns

- **Platonic architecture:** Designing the perfect interface before any implementation.
- **Abstraction astronautics:** Generic code with one caller.
- **Premature DRY:** Removing duplication before you understand variation.
- **Spike hell:** Endless prototyping with no harvest.
- **Fossilized spikes:** Prototype code promoted to production without refactoring.
- **No question:** Coding because it feels productive, not because it answers a design unknown.
- **Attachment:** Refusing to delete code that has already taught you what you need.

---

## Chapter 12: Exercises and Case Studies

### Exercise 1: Top-Down Skeleton

Choose a small tool you want to build. Write the main flow in pseudocode. Stub every lower-level piece. Run the skeleton with fake data. What boundaries did you discover?

### Exercise 2: Bottom-Up Concrete Cases

Pick one stub. Implement it for one concrete case. Then a second. Then a third. What pattern emerges? What abstraction is now obvious?

### Exercise 3: UNIX Pipeline

Redesign your tool as a pipeline of small components. Each component does one thing. Each communicates via plain data. Spike one component independently.

### Exercise 4: Harvest Note

Write a harvest note for your last spike. Include the question, the concrete cases, the invariant, the variation, and the decision.

### Exercise 5: Delete with Gratitude

Find a piece of spike code you have been keeping “just in case.” Extract the lesson. Delete the code. Notice how the project feels lighter.

---

# Appendices

## Appendix A: Glossary

- **Spike:** A timeboxed concrete experiment to answer a design question.
- **Top-down spike:** A spike that organizes the work by creating a skeleton with stubs.
- **Bottom-up spike:** A spike that fleshes out the skeleton by implementing concrete cases.
- **Harvest:** The act of extracting the design lesson from a spike.
- **Non-attachment:** The discipline of treating code as disposable scaffolding.
- **UNIX philosophy:** Small, sharp, composable tools with clear interfaces and plain data.
- **Existence precedes essence:** The abstraction emerges from concrete implementations, not the other way around.

## Appendix B: Spike Card

```text
Question:
Concrete case 1:
Concrete case 2:
Friction / duplication:
Invariant:
Variation:
Candidate abstraction:
What the compiler forced:
Delete or keep:
```

## Appendix C: Harvest Note

```text
Spike name:
Question:
What we tried:
What happened:
What repeated:
What varied:
Candidate abstraction:
Decision:
Follow-up spikes:
```

## Appendix D: Skeleton Map

```text
[readInput] -> [parse] -> [process] -> [format] -> [writeOutput]
     |            |           |           |            |
   stubbed      stubbed     stubbed     stubbed      stubbed
```

Update as stubs are replaced.

## Appendix E: UNIX Checklist

- Does it do one thing well?
- Can it be composed with other components?
- Is the interface plain and explicit?
- Is policy separated from mechanism?
- Can I spike this part independently?
- Is silence maintained when there is nothing to say?

---

# Conclusion

The next level of software design is not more abstraction. It is faster, more disciplined loops between concrete and abstract.

You write the wrong thing precisely. You compile it. You watch what happens. Then you let the right abstraction emerge.

Use top-down spikes to organize the work. Use bottom-up spikes to flesh out the skeleton. Use the UNIX philosophy to keep parts small, sharp, and composable. Stay non-attached to disposable code. Keep the lesson, delete the scaffold.

> **Existence first. Essence earned. Spike, harvest, delete, repeat.**

