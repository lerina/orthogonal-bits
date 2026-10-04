# THE DESCENT DRILLS — PART ONE

*From the training curriculum built around the RUSTCORE Field Manual. Not the philosophy — the drills. The part apprentices actually run, over and over, until the six-step loop stops being something they think about and starts being something they just do.*

---

> The Field Manual tells you what the loop is: Question, Query, Answer, Narrow, Trace, Verify. It doesn't teach you how to move your hands. That's what the Descent Drills are for — the actual repetitions, built specifically around Rust source, because Rust source is what most of the Line eventually has to get through, and because a codebase turns out to be a remarkably good stand-in for the thing every apprentice is really training toward.
>
> Every runner who's been through this curriculum describes the same shift happening somewhere around the middle of it. A source tree stops looking like a wall of text and starts looking like exactly what the opening lecture insists it already is: a network of definitions, calls, types, modules, dependencies, transformations, and control flow — not prose, and never to be read like prose again. Four parts. This is the first.

---

# Decomposing Source Code with grep and ripgrep

## A Text-First Method for Code Exploration

Source code is not prose. Prose is normally consumed linearly.

Code is better understood as a network of:

* definitions
* calls
* types
* modules
* dependencies
* data transformations
* control flow

For a programmer using a screen reader, this distinction matters.

Visual programmers can scan a file and immediately notice indentation, symbols, declarations, repeated names, and structural landmarks.

A text-first workflow needs another way to create those landmarks.

`grep` and `ripgrep` provide that mechanism.

The basic idea is:

```text
MAP then SEARCH then LOCATE then TRACE then READ then PARAPHRASE
```

The goal is not to read less code.

The goal is to find the **right code to read deeply**.

---

> **Drill note, opening session:** apprentices are told, before touching a single command, that this sentence is the only part of the whole curriculum they're required to memorize word for word: *the goal is not to read less code, the goal is to find the right code to read deeply.* Everything below it is technique. This line is the reason the technique exists at all.

---

## part 1

# 1. The Codebase as a Map

Before reading implementation details, establish the shape of the project.

For a Rust project:

```bash
rg --files -t rust
```

Find all project files:

```bash
rg --files
```

Find Rust source files:

```bash
rg --files -g '*.rs'
```

Find Markdown documentation:

```bash
rg --files -g '*.md'
```

Find configuration files:

```bash
rg --files | rg '(^|/)(Cargo\.toml|Cargo\.lock)$'
```

Find likely entry points:

```bash
rg --files | rg '(^|/)(main|lib)\.rs$'
```

Find tests:

```bash
rg --files | rg '(^|/)tests?/'
```

This gives you a structural map without opening every file.

---

# 2. `rg --files` Is Your Directory Reconnaissance Tool

You usually do not need a separate directory-tree program.

Start with:

```bash
rg --files
```

Then narrow the result.

Rust:

```bash
rg --files -g '*.rs'
```

Tests:

```bash
rg --files | rg '(^|/)(tests?|benches?)/'
```

Documentation:

```bash
rg --files -g '*.md'
```

Configuration:

```bash
rg --files | rg '(Cargo\.toml|Cargo\.lock|\.toml)$'
```

Hidden files can be included when needed:

```bash
rg --files --hidden
```

Remember that ripgrep normally respects ignore rules and skips hidden files and binary files. That is usually desirable for code exploration.

---

> **Drill note:** this is the first exercise where the room usually splits. Some apprentices try to open the file browser out of old habit before they've even finished typing the command — a reflex from every other kind of work they've ever done. Instructors let it happen once, without comment, and just wait. The point isn't the correction. It's letting someone feel, firsthand, how much slower the old habit is next to a single line that already answered the question.

---

# 3. Find the Entry Point

For Rust:

```bash
rg --files | rg '(^|/)main\.rs$'
```

and:

```bash
rg --files | rg '(^|/)lib\.rs$'
```

A binary crate normally has a `main.rs`. A library crate normally has a `lib.rs`.
A workspace may contain several crates, so search the whole repository rather than assuming there is one entry point.

Find all of them:

```bash
rg --files | rg '/(main|lib)\.rs$'
```

Then inspect the definitions:

```bash
rg -n '^\s*fn main\s*\(' -t rust
```

```bash
rg -n '^\s*(pub\s+)?fn ' -t rust
```

---

# 4. Find the Project Configuration

For a Rust project:

```bash
rg --files | rg '(^|/)Cargo\.toml$'
```

Then search it:

```bash
rg -n '^\[|^name\s*=|^version\s*=|^edition\s*=|^dependencies' Cargo.toml
```

Find dependency declarations:

```bash
rg -n '^\w[\w-]*\s*=' Cargo.toml
```

For a workspace:

```bash
rg -n '^\[workspace\]|members\s*=' Cargo.toml
```

The purpose is not to understand the whole configuration immediately.

The purpose is to establish:

```text
What is this project?
What crates exist?
What are its major dependencies?
```

---

> **Drill note:** apprentices are told to treat `Cargo.toml` as a kind of dossier before they ever meet the actual code — three questions, answered from a single file, before a single function definition has been read. Instructors on the Line like to point out, dryly, that this is the exact same habit the intake process runs on every apprentice who shows up wanting to learn it. Name. Dependencies. What you actually are, before anyone reads a line of what you can do.

---

# 5. Build a Structural Outline of a Rust File

Once you have a file, do not immediately read every line.

First ask:

> What does this file contain?

Search the major Rust declarations:

```bash
rg -n '^\s*(pub\s+)?(use|mod|struct|enum|trait|impl|fn)\b' src/main.rs
```

This gives you a structural outline.

You can make the search more specific.

Functions:

```bash
rg -n '^\s*(pub\s+)?fn\s+\w+' src/main.rs
```

Structs:

```bash
rg -n '^\s*(pub\s+)?struct\s+\w+' src/main.rs
```

Enums:

```bash
rg -n '^\s*(pub\s+)?enum\s+\w+' src/main.rs
```

Traits:

```bash
rg -n '^\s*(pub\s+)?trait\s+\w+' src/main.rs
```

Implementations:

```bash
rg -n '^\s*impl\b' src/main.rs
```

Modules:

```bash
rg -n '^\s*(pub\s+)?mod\s+\w+' src/main.rs
```

Imports:

```bash
rg -n '^\s*use\s+' src/main.rs
```

---

# 6. Search the Whole Rust Project

Once you understand one file, widen the search.

Find all function definitions:

```bash
rg -n '^\s*(pub\s+)?fn\s+\w+' -t rust
```

Find all structs:

```bash
rg -n '^\s*(pub\s+)?struct\s+\w+' -t rust
```

Find all enums:

```bash
rg -n '^\s*(pub\s+)?enum\s+\w+' -t rust
```

Find all traits:

```bash
rg -n '^\s*(pub\s+)?trait\s+\w+' -t rust
```

Find all implementations:

```bash
rg -n '^\s*impl\b' -t rust
```

This creates a machine-readable architectural outline.

---

# 7. Find a Definition

Suppose you encounter:

```text
process_data
```

Search for its definition:

```bash
rg -n '^\s*(pub\s+)?fn\s+process_data\s*\(' -t rust
```

For a struct:

```bash
rg -n '^\s*(pub\s+)?struct\s+User\b' -t rust
```

For an enum:

```bash
rg -n '^\s*(pub\s+)?enum\s+State\b' -t rust
```

For a trait:

```bash
rg -n '^\s*(pub\s+)?trait\s+Processor\b' -t rust
```

This is better than searching only for the name because the declaration keyword gives the search structural meaning.

---

> **End of Part One.** Seven exercises, and not one of them has asked an apprentice to read a function body yet — on purpose. The instructors are explicit about this being deliberate, not an oversight: you don't get to touch implementation until you can already answer, cold, what a project *is* before you've read a single line of what it *does*. Part Two is where that finally changes — definitions give way to uses, and the drills stop being about a single file and start being about how everything in a project actually talks to everything else.
