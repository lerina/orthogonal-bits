# THE DESCENT DRILLS — PART THREE

*Continued from Part Two. The graph-building is done. Now the curriculum turns toward what's worth being suspicious of.*

---

> Parts One and Two teach an apprentice to find what's true about a codebase — what's defined, what's used, what calls what. Part Three is where the tone shifts, deliberately. Instructors on the Line describe this stretch as the point where a runner stops being a cartographer and starts being something closer to an inspector: still asking questions, still following the same loop, but now the questions are aimed at the places a project would rather you didn't look too closely.

---

## Part 3

# 21. Search for Risky Patterns

`rg` is useful for finding things that deserve review.

For example:

```bash
rg -n '\b(unwrap|expect|panic!)' -t rust
```

Find TODOs:

```bash
rg -n 'TODO|FIXME|XXX' -t rust
```

Find unsafe blocks:

```bash
rg -n '\bunsafe\b' -t rust
```

Find shell-command execution:

```bash
rg -n '\b(Command|std::process::Command)\b' -t rust
```

Find SQL-related code:

```bash
rg -ni '\b(sql|query|execute)\b' -t rust
```

These searches identify **places worth reviewing**.

They do not prove that the code contains a defect.

---

> **Drill note:** instructors are strict about that last line, to the point of making apprentices repeat it back before moving on. A hit on `unsafe` or `unwrap` isn't a verdict, it's a lead — and a runner who treats every hit as a confirmed problem burns out fast, chasing ghosts instead of actual weak points. The discipline here is holding suspicion and patience in the same hand.

---

# 22. Search for Tests

Find test modules:

```bash
rg -n '#\[cfg\(test\)\]' -t rust
```

Find test functions:

```bash
rg -n '#\[test\]' -t rust
```

Find tests related to a symbol:

```bash
rg -n -C 3 'process_data' -g '*.rs'
```

Find integration-test files:

```bash
rg --files | rg '(^|/)tests?/'
```

A useful question is:

> Where is this behavior tested?

That question often reveals how the author expects the code to behave.

---

# 23. Search Documentation and Comments

Find documentation comments:

```bash
rg -n '^\s*///' -t rust
```

Find ordinary comments:

```bash
rg -n '^\s*//' -t rust
```

Find documentation for a specific function:

```bash
rg -n -B 5 '^\s*(pub\s+)?fn\s+process_data' -t rust
```

Find references to a concept:

```bash
rg -ni 'authentication|authorization' -t rust
```

Search code and documentation together:

```bash
rg -ni 'authentication' -g '*.rs' -g '*.md'
```

This can reveal both implementation and the project's explanation of that implementation.

---

> **Drill note:** sections 22 and 23 are taught back to back on purpose. A test tells you what the code actually does under specific conditions. A comment tells you what the author *believed* it did. Instructors call the gap between those two things one of the most reliable places to find where a project quietly went wrong without anyone noticing yet.

---

# 24. Search Attributes and Macros

Rust attributes are important structural landmarks.

Find attributes:

```bash
rg -n '^\s*#\[' -t rust
```

Find derive declarations:

```bash
rg -n '#\[derive\(' -t rust
```

Find tests:

```bash
rg -n '#\[test\]' -t rust
```

Find conditional compilation:

```bash
rg -n '#\[cfg' -t rust
```

Find macro definitions:

```bash
rg -n 'macro_rules!' -t rust
```

Find macro invocations with a known name:

```bash
rg -n '\bprintln!\s*\(' -t rust
```

---

# 25. Search by Relationship

A powerful way to use `rg` is to ask relationship questions.

Instead of:

> Show me everything.

Ask:

> Where is this defined?

```bash
rg -n '^\s*(pub\s+)?fn\s+NAME\s*\(' -t rust
```

> Where is it used?

```bash
rg -n -w 'NAME' -t rust
```

> Where is it imported?

```bash
rg -n '^\s*use\s+.*\bNAME\b' -t rust
```

> Where is it tested?

```bash
rg -n -C 3 '\bNAME\b' -g '*.rs'
```

> What calls it?

```bash
rg -n '\bNAME\s*\(' -t rust
```

> What documentation mentions it?

```bash
rg -ni '\bNAME\b' -g '*.md'
```

This is the heart of code interrogation.

---

> **Drill note:** this section is treated as the exam, informally, even though nobody on the Line calls it that officially. Every question above gets asked about a symbol the instructor picks on the spot, and the apprentice has to produce the query without being handed the template first. Runners who've been through it say it's the first moment the drills stop feeling like an exercise and start feeling like the actual job.

---

# 26. `grep` Still Has a Place

`rg` should be the primary tool for repository exploration.

`grep` remains useful for simple local filtering.

For one file:

```bash
grep -n 'error' build.log
```

Case-insensitive:

```bash
grep -in 'error' build.log
```

With context:

```bash
grep -C 3 'error' build.log
```

Pipeline filtering:

```bash
some-command | grep 'pattern'
```

The principle is simple:

```text
Simple local filter then    grep
Repository interrogation then    rg
```

There is no reason to introduce a larger tool when `grep` or `rg` already solves the problem.

---

# 27. What `rg` Should Not Try to Do

`rg` is a search tool, not a Rust parser.

It is excellent for:

* finding declarations
* finding names
* finding calls
* finding imports
* finding attributes
* finding tests
* finding patterns
* locating landmarks
* comparing textual patterns
* narrowing the code you need to read

It is not reliable for:

* determining the exact end of an arbitrarily nested Rust function
* constructing a complete semantic call graph
* understanding type resolution
* proving that two pieces of code are semantically equivalent
* replacing the Rust compiler
* replacing an AST parser

When the question becomes semantic rather than textual, switch tools.

But make that switch deliberately.

---

> **Drill note:** apprentices are told that section 27 is the one they're allowed to disagree with instructors about — encouraged to, even. Knowing the edge of a tool's competence matters more than reciting its strengths, and an apprentice who can't say where `rg` runs out of authority hasn't actually learned it yet, no matter how fast they've gotten with it.

---

# 28. The Screen-Reader Workflow

A practical workflow is:

### Step 1: Map

```bash
rg --files
```

### Step 2: Restrict

```bash
rg --files -t rust
```

### Step 3: Find landmarks

```bash
rg -n '^\s*(pub\s+)?(fn|struct|enum|trait|impl|mod)\b' -t rust
```

### Step 4: Find the entry point

```bash
rg -n '^\s*fn main\s*\(' -t rust
```

### Step 5: Read the startup sequence

```bash
rg -n -A 30 '^\s*fn main\s*\(' -t rust
```

### Step 6: Find the next important function

```bash
rg -n '^\s*(pub\s+)?fn\s+NAME\s*\(' -t rust
```

### Step 7: Find its uses

```bash
rg -n -w 'NAME' -t rust
```

### Step 8: Read the relevant code in Helix.

Here we connect **regex search** with **Helix's selection-first model and textobjects**. 
The official Helix documentation describes textobjects exactly as structured pieces of text that can be selected and operated on, including functions, types, parameters, comments, and tests. ([Helix][1])


Once `rg` has located the relevant Rust code, open the file in Helix.

The goal is not to read the entire file from top to bottom.

The goal is:

```text
SEARCH then    LOCATE then    SELECT then    READ then    TRACE
```

`rg` is excellent for finding **where** something occurs across the repository.

Helix is where you inspect the actual code and work with structured selections.

see more in [part 4](compile-curriculum-part04.md)

---


### Step 9: Repeat.

This creates a controlled descent into the codebase.

---

> **End of Part Three.** Eight steps, and the last word of the whole sequence is *repeat* — no closing ceremony, no graduation line, the same refusal to wrap things up neatly that runs through everything this curriculum was built from. Instructors don't add one either. Part Four, wherever it eventually surfaces, is understood on the Line to be less a continuation than an open door: the drills end, and the actual codebases don't.
