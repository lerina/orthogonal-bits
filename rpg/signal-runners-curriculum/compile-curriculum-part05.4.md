# THE DESCENT DRILLS — Skill Building (IV)

*Continued from Skill Building (iii). The last installment — of Skill Building, and of the curriculum it belongs to.*

---

> Everything up to here has been drills in isolation: one tool, one distinction, one correction at a time. This installment is where the appendix — and the curriculum built around it — finally stops teaching pieces and lays out the whole shape at once. Instructors don't add much of their own framing to this last stretch. They don't need to. The material does the closing argument on its own.

---

# 12. Regex Differences Between Helix and ripgrep

Helix and ripgrep both use the Rust regex ecosystem, but they are different implementations and interfaces.

Helix performs regex searches inside the editor.

ripgrep performs regex searches across files and directories.

Helix's document search uses `regex-cursor`, which was developed to provide regex searching over Helix's rope-based document representation. It is based on the Rust regex ecosystem rather than being a direct use of the ordinary `regex` crate API. ([Helix][1])

Therefore:

```text
Regex syntax
```

and:

```text
Tool command
```

should be learned separately.

For example:

```text
-F
```

is a ripgrep command-line option.

It is not part of regex syntax.

Likewise:

```text
/pattern
```

is a Helix command.

It is not something you type at a shell prompt.

And:

```bash
rg 'pattern'
```

is a ripgrep invocation.

It is not a Helix command.

### Do not assume:

1. A command available in one tool exists in the other.
2. A feature from another regex engine exists in Rust regex.
3. Regex syntax and editor commands are the same thing.

---

# 13. Practical Regex Patterns for Code Navigation

## Function definitions

```regex
^\s*(pub\s+)?(async\s+)?fn\s+\w+
```

Likely Rust function declarations.

## Trait implementations

```regex
impl\s+\w+\s+for\s+\w+
```

For example:

```rust
impl Display for User
```

This is only a rough pattern because Rust permits more complicated implementation syntax.

## Match statements

```regex
match\s+\w+\s*\{
```

A rough search for match statements with a simple word as the scrutinee.

## Calls followed by `?`

```regex
\w+\([^)]*\)\?
```

A rough search for a simple function call followed by `?`.

It does not correctly model nested parentheses.

## Result

```regex
\bResult\b
```

Find `Result` as a whole word.

## Possible commented-out Rust

```regex
^\s*//.*(fn|impl|pub|struct|enum|trait|match|if|for|while)
```

This is a heuristic for comments that may contain Rust-like code.

It is not proof that the comment contains executable or previously executable code.

### The rule

> **Use regex for navigation and approximate structural searches. Use a parser when you need actual program structure.**

---

# 14. The Master Regex Rules

> 1. Regex is a query language for text.
>
> 2. `^` and `$` are line-oriented anchors in ordinary line-based searching.
>
> 3. `\A` and `\z` refer to the beginning and end of the regex input; their practical scope depends on the tool.
>
> 4. `\d`, `\w`, and `\s` are Unicode-aware in Rust's regex syntax by default.
>
> 5. Use explicit ASCII classes such as `[0-9]` when ASCII is what you mean.
>
> 6. Use `\b` for word boundaries.
>
> 7. Use `[abc]` and `[^abc]` for character classes.
>
> 8. Use `*`, `+`, `?`, and `{n,m}` to control repetition.
>
> 9. Use parentheses for capture groups.
>
> 10. Use `|` for alternation.
>
> 11. Rust regex does not support general lookahead or lookbehind.
>
> 12. Helix uses `/pattern` for forward regex search and `?pattern` for reverse search.
>
> 13. ripgrep interprets patterns as regex by default.
>
> 14. Use `rg -F` when you want literal text rather than regex.
>
> 15. Quote regex patterns in the shell.
>
> 16. Regex searches text. It does not understand Rust program structure.
>
> 17. Use ast-grep when the question is syntactic.
>
> 18. Use rust-analyzer or compiler-backed tools when the question is semantic.
>
> 19. Start with a simple pattern and add complexity only when the simple pattern is insufficient.

---

> **Drill note:** apprentices are handed these nineteen rules as a single page, deliberately, without commentary. Instructors don't explain them line by line the way earlier sections got explained. By this point in the curriculum the expectation has flipped — a runner should already recognize every rule on this page as something they've personally made the mistake of forgetting, at least once, somewhere in Parts One through Four (iii).

---

# 15. The Regex Debugging Mantra

When a regex does not match what you expect, ask:

* Did I escape special characters correctly?
* Did I use `|` for alternation?
* Did I accidentally write `\|`?
* Is the pattern anchored with `^` or `$` when it needs to be?
* Are my quantifiers correct?
* Do I need a word boundary with `\b`?
* Is my character class correct?
* Am I expecting ASCII behavior from a Unicode-aware class?
* Am I confusing Helix commands with ripgrep options?
* Did the shell interpret part of my pattern before ripgrep received it?
* Does the pattern need to span multiple lines?
* Would a simpler pattern answer the question?
* Am I trying to parse Rust structure with a text-search tool?

The single most important question is:

> **What exact text do I expect to match, and what exact text does my pattern actually describe?**

---

# 16. The Grand Unified Model

Think of the tools as three levels.

## Level 1 — Regex

Regex describes:

> **What text pattern am I looking for?**

Example:

```regex
^\s*(pub\s+)?fn\s+\w+
```

## Level 2 — ripgrep

ripgrep asks:

> **Where does this text pattern occur across the repository?**

Example:

```bash
rg '^\s*(pub\s+)?fn\s+\w+' -g '*.rs'
```

## Level 3 — Helix

Helix asks:

> **Where is this match inside the file, and what do I want to do with the selection?**

Example:

```text
/^\s*(pub\s+)?fn\s+\w+
```

Then:

```text
n
```

to move through matches.

## Level 4 — ast-grep / rust-analyzer

When the question becomes:

> **What does this code actually mean structurally or semantically?**

regex is no longer necessarily the right abstraction.

Use:

```text
ast-grep
```

for syntax-aware structural searches.

Use:

```text
rust-analyzer
```

for language-semantic information such as definitions, references, types, implementations, and other compiler-aware relationships.

---

# 17. The Working Hierarchy

For everyday Rust code archaeology, keep the escalation simple:

```text
literal search
    ↓
regex
    ↓
rg
    ↓
Helix
    ↓
ast-grep
    ↓
rust-analyzer / compiler
```

Start with the cheapest question.

For example:

> Where does `process_data` appear?

```bash
rg -n 'process_data' -g '*.rs'
```

If you need a likely definition:

```bash
rg -n '^\s*(pub\s+)?(async\s+)?fn\s+process_data\b' -g '*.rs'
```

If you need surrounding source:

```bash
rg -n -A 30 '^\s*(pub\s+)?(async\s+)?fn\s+process_data\b' -g '*.rs'
```

If you need to understand the actual syntax:

```bash
ast-grep outline src/
```

If you need actual symbol relationships:

```text
rust-analyzer / LSP
```

### Final principle

> **Use the simplest tool that can answer the question reliably.**

Regex is powerful because it turns text into queries.

ripgrep makes those queries repository-wide.

Helix lets you interrogate and manipulate the matches inside the source.

ast-grep understands syntax.

rust-analyzer understands Rust semantics.

That division of labor keeps the workflow small while still giving you a path from simple textual reconnaissance to genuine code understanding.

---

*End*

---

[1]: https://helix-editor.com/news/release-24-03-highlights/ "Release 24.03 Highlights | Helix"
[2]: https://docs.helix-editor.com/master/from-vim.html "Migrating from Vim"
[3]: https://ast-grep.github.io/reference/cli/outline "ast-grep outline | ast-grep"
[4]: https://docs.rs/regex/latest/regex/ "regex - Rust"
[5]: https://ast-grep.github.io/guide/quick-start "Quick Start | ast-grep"
[6]: https://ast-grep.github.io/reference/cli "Command Line Reference | ast-grep"
[7]: https://docs.helix-editor.com/master/commands.html "Commands"
[8]: https://docs.rs/crate/ripgrep/latest/source/GUIDE.md "ripgrep 15.2.0 - Docs.rs"

---

> The appendix ends the same way every other part of this curriculum has ended, on purpose: no summary paragraph, no send-off, just *End*, sitting alone where a conclusion would normally go. Runners who've been through all four parts, in every one of their installments, tend to agree that this is the last lesson hiding inside the formatting itself — the discipline was never building toward a finish line. It was building toward a habit that keeps running long after the document handing it to you has stopped.
>
> Four parts Skill Building. Regex, source, structure, and the quiet, repeated insistence that none of it is a substitute for asking the next honest question. That's the whole curriculum. What a runner does with it, from here, was never going to fit on the page.
