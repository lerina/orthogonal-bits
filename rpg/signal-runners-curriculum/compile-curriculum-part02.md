# THE DESCENT DRILLS — PART TWO

*Continued from Part One. Definitions are done. Now the drills turn toward everything a definition actually touches.*

---

> Part One ends deliberately at the edge of a function body, with instructors refusing, on principle, to let an apprentice read implementation before they can answer what a project is. Part Two is where that restriction finally lifts — and it lifts all at once, because the moment you're allowed to look at what something *does*, you're also expected to trace everywhere it's *used*, *called*, *imported*, and *broken*, in the same sitting. Runners who've been through it describe this stretch of the curriculum as the one that actually teaches the shape of a codebase, rather than just its outline.

---

## Part 2

# 8. Find Uses

After locating a definition, search for its uses.

For a type:

```bash
rg -n -w 'User' -t rust
```

For a function:

```bash
rg -n -w 'process_data' -t rust
```

For an enum:

```bash
rg -n -w 'State' -t rust
```

`-w` restricts the match to a whole word. This is often preferable to manually adding `\b` around a name.

The result gives you a first approximation of the symbol's neighborhood in the codebase.

---

# 9. Distinguish Definition from Use

Suppose the symbol is:

```text
process_data
```

Find the definition:

```bash
rg -n '^\s*(pub\s+)?fn\s+process_data\s*\(' -t rust
```

Find calls:

```bash
rg -n '\bprocess_data\s*\(' -t rust
```

The first search answers:

> Where is it defined?

The second asks:

> Where is it invoked?

That distinction is fundamental to code archaeology.

---

> **Drill note:** instructors call sections 8 and 9 the "two-question pair," and they refuse to let an apprentice move on until they can explain, unprompted, why the two searches are different questions rather than the same question twice. It's a smaller version of a distinction the Field Manual spends much longer making: evidence isn't the same shape twice just because it came from the same symbol.

---

# 10. Trace Imports

For a module:

```bash
rg -n '^\s*use\s+crate::models' -t rust
```

For a particular type:

```bash
rg -n '^\s*use\s+.*\bUser\b' -t rust
```

For an external crate:

```bash
rg -n '^\s*use\s+tokio::' -t rust
```

This tells you which parts of the codebase depend on that namespace.

---

# 11. Trace Execution from `main`

Start with:

```bash
rg -n -A 20 '^\s*fn main\s*\(' -t rust
```

The `-A 20` means:

> Show the matching line and 20 lines after it.

Look for the first significant function call.

Suppose you find:

```rust
run_application();
```

Search for its definition:

```bash
rg -n '^\s*(pub\s+)?fn\s+run_application\s*\(' -t rust
```

Then inspect it:

```bash
rg -n -A 30 '^\s*(pub\s+)?fn\s+run_application\s*\(' -t rust
```

Find the next significant call.

Repeat.

The resulting chain might look like:

```text
main then run_application then load_config then build_runtime then process_request then write_output
```

That is an execution-path map.

---

> **Drill note:** this exercise is the one apprentices tend to remember longest, because it's the first time the curriculum asks them to build something and keep building it, call after call, with no fixed endpoint given in advance. Instructors let the chase run as long as it needs to. The lesson isn't the specific chain any one apprentice ends up with. It's the experience of realizing, partway through, that they're no longer following instructions — they're the ones deciding where the trace goes next.

---

# 12. Use Context Instead of Extracting Functions

The original method used `awk` to try to extract a complete Rust function.

That approach is unreliable because Rust functions can contain nested blocks.

Instead, start with context:

```bash
rg -n -A 30 'fn process_data' src/main.rs
```

If that is not enough:

```bash
rg -n -A 80 'fn process_data' src/main.rs
```

If the function is large, locate its beginning and then open the file at that line in Helix.

The important distinction is:

```text
rg finds the landmark.
Helix performs the deep reading.
```

`rg` does not need to become a parser.

---

# 13. Interrogate a Function

Suppose you have located `process_data`.

Search for branches:

```bash
rg -n -A 80 'fn process_data' src/main.rs |
    rg -n '\b(if|else|match)\b'
```

Search for loops:

```bash
rg -n -A 80 'fn process_data' src/main.rs |
    rg -n '\b(for|while|loop)\b'
```

Search for returns:

```bash
rg -n -A 80 'fn process_data' src/main.rs |
    rg -n '\breturn\b'
```

Search for error handling:

```bash
rg -n -A 80 'fn process_data' src/main.rs |
    rg -n '\b(unwrap|expect|panic|Result|Error)\b'
```

This is useful for reconnaissance.

For exact understanding, switch back to line-by-line reading.

---

# 14. Search Assignments

Find simple local declarations:

```bash
rg -n '\blet\s+\w+' -t rust
```

Find mutable declarations:

```bash
rg -n '\blet\s+mut\s+\w+' -t rust
```

Find assignments:

```bash
rg -n '\b\w+\s*=' -t rust
```

The last pattern is deliberately broad.

It will also match things such as `==`, `=>`, and parts of other expressions.

So use it as a reconnaissance search, not as a precise parser.

For a specific variable:

```bash
rg -n '\bresult\b' -t rust
```

Then inspect the surrounding lines.

---

> **Drill note:** this is the one section where instructors actively want an apprentice's first attempt to misfire. The broad assignment pattern is left in the curriculum precisely because it over-matches, and watching someone wade through the noise it produces teaches the lesson faster than any warning could. A tool that answers too broadly is still telling you something — usually that you asked the wrong-sized question.

---

# 15. Search Function Calls

A rough call search can be useful:

```bash
rg -n '\b\w+\s*\(' -t rust
```

But this produces many false positives.

A better approach is to search for a known function:

```bash
rg -n '\bprocess_data\s*\(' -t rust
```

This is one reason code archaeology should proceed from known landmarks.

Instead of trying to extract the entire call graph automatically:

```text
find symbol then find definition then find uses then follow important calls
```

---

# 16. Build a Call Graph Manually

For each important function:

### Find its definition

```bash
rg -n '^\s*(pub\s+)?fn\s+process_data\s*\(' -t rust
```

### Find its callers

```bash
rg -n '\bprocess_data\s*\(' -t rust
```

### Find the calls it makes

Once you are reading the function, identify important calls and search each one.

For example:

```bash
rg -n '^\s*(pub\s+)?fn\s+load_config\s*\(' -t rust
```

Then:

```bash
rg -n '\bload_config\s*\(' -t rust
```

You are building the graph incrementally.

This is often more useful than attempting to generate a complete automated call graph.

---

# 17. Trace a Type

Suppose you encounter:

```rust
User
```

Find its definition:

```bash
rg -n '^\s*(pub\s+)?struct\s+User\b' -t rust
```

Find its uses:

```bash
rg -n -w 'User' -t rust
```

Find fields:

```bash
rg -n -A 20 '^\s*(pub\s+)?struct\s+User\b' -t rust
```

Then investigate the important methods:

```bash
rg -n '^\s*(pub\s+)?fn\s+' -t rust
```

At this point you can ask:

```text
Where is User defined?
Where is it constructed?
Where is it passed?
Where is it modified?
Where is it consumed?
```

Those questions reveal the type's role in the system.

---

> **Drill note:** apprentices are told, at this point in the curriculum and not before, that these five questions are the same five questions the Field Manual eventually applies to an entire RUSTCORE checkpoint rather than a single struct. Nobody explains this the first time through. It's meant to be noticed later, on its own, usually the first time a runner is standing in front of something much bigger than a `User` type and realizes they already know exactly how to start.

---

# 18. Trace a Module

Suppose the project contains:

```text
src/models.rs
```

Find module declarations:

```bash
rg -n '^\s*(pub\s+)?mod\s+models\b' -t rust
```

Find imports:

```bash
rg -n '^\s*use\s+.*models' -t rust
```

Find references:

```bash
rg -n '\bmodels\b' -t rust
```

The three searches answer different questions:

```text
Where is the module declared?
Where is it imported?
Where is it referenced?
```

---

# 19. Use Match Counts as a Lead, Not a Metric

`rg -c` counts matching lines per file:

```bash
rg -c -w 'User' -t rust
```

This can help prioritize exploration.

A file with many references to `User` may deserve investigation.

But match count is **not** a reliable measure of architectural importance or complexity.

For individual matches rather than matching lines, use:

```bash
rg --count-matches -w 'User' -t rust
```

Use the result as a clue.

Then inspect the actual matches.

---

# 20. Debugging Interrogation

When an error occurs, start with the exact error text.

```bash
rg -n 'error: expected' build.log
```

If the compiler gives you:

```text
src/parser.rs:87
```

go directly to that location in Helix.

Then search the surrounding code for relevant symbols.

For example:

```bash
rg -n -C 10 'problematic_variable' src/parser.rs
```

Search for mutations:

```bash
rg -n '\bproblematic_variable\b' src/parser.rs
```

Search for error-handling operations:

```bash
rg -n '\b(unwrap|expect|panic|Err|Error)\b' src/parser.rs
```

The debugging loop becomes:

```text
error then location then surrounding code then relevant variable then definition then mutations then callers
```

---

> **End of Part Two.** A full manual chase, start to finish — definition to use, use to call, call to graph, type to module, and finally a real compiler error run back through the exact same instincts. Nothing here has needed a new idea since Part One. Same loop, wider net, every time. Part Three is where the curriculum stops asking apprentices to only find what's correct, and starts asking them to find what's dangerous.
