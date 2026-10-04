# THE DESCENT DRILLS — Skill Building (II)

*Continued from Skill Building (i). Character classes and quantifiers are behind us. Now the appendix turns toward position, capture, and the first honest admission of where regex runs out of road.*

---

> Apprentices are told, going into this installment, to pay closest attention to section 7 — not because it's the hardest technically, but because it's the first place in the whole curriculum where the Field Manual openly says *this tool cannot do that, stop asking it to.* Instructors treat a runner's reaction to that section as diagnostic. The ones who get frustrated by the limitation usually aren't ready for what comes after it. The ones who nod and reach for the next tool already are.

---

# 5. Anchors and Boundaries

Common anchors and boundaries include:

```text
^       start of line
$       end of line
\A      start of the regex input
\z      end of the regex input
\b      word boundary
\B      non-boundary
```

The exact practical scope of `\A` and `\z` depends on how the application supplies text to the regex engine.

For ripgrep, ordinary searches operate on lines unless multiline mode is explicitly requested. Therefore, for everyday Rust-code searching, `^` and `$` are usually the anchors you want.

A useful Rust-oriented pattern is:

```regex
^\s*fn\s+\w+
```

This looks for a line that:

1. starts at the beginning of the line,
2. may contain leading whitespace,
3. contains `fn`,
4. contains whitespace,
5. contains a word-like function name.

It is a **heuristic**, not a Rust parser.

For example, it can miss or mishandle syntax such as:

```rust
pub async fn ...
```

because the pattern does not account for every possible modifier.

A better rough search is:

```regex
^\s*(pub\s+)?(async\s+)?fn\s+\w+
```

Even that is still only a textual approximation.

### Important principle

> **Regex recognizes text patterns. It does not understand Rust syntax.**

---

# 6. Capture Groups

Parentheses create capture groups:

```regex
(fn)\s+(\w+)
```

Given:

```text
fn main
```

the groups are:

```text
Capture 1: fn
Capture 2: main
```

Named capture groups use:

```regex
(?P<n>\w+)
```

Capture groups are useful when a tool exposes the captured portions for extraction or replacement.

However, the presence of capture groups in the regex syntax does **not** mean that every tool provides capture-based replacement.

ripgrep is primarily a search tool. It does not perform in-place replacement.

Helix provides a selection-based editing workflow rather than a traditional regex substitution command.

### The rule

> **A regex can capture text, but the tool decides what you can do with those captures.**

---

# 7. An Important Gap: No Look-Around

The Rust regex syntax does not support general lookahead or lookbehind.

These are unavailable:

```text
(?=pattern)       positive lookahead
(?!pattern)       negative lookahead
(?<=pattern)      positive lookbehind
(?<!pattern)      negative lookbehind
```

For example:

```text
(?<!foo)bar
```

is not valid Rust-regex syntax.

When you need this kind of contextual logic, practical alternatives include:

1. Reformulate the pattern.
2. Search in multiple passes.
3. Search a broader pattern and inspect the surrounding text.
4. Pipe one `rg` search into another `rg`.
5. Use `grep -v` when simple exclusion is enough.
6. Use a syntax-aware tool such as ast-grep or rust-analyzer when the question is structural rather than textual.

### The rule

> **Know which regex engine you are using. Regex syntax is not universal.**

---

> **Drill note:** instructors point out that item 6 on that list — reach for ast-grep — isn't a last resort tacked on at the end. It's the entire reason the next section exists. The Field Manual doesn't apologize for the gap in look-around support and doesn't try to route around it with an uglier regex either. It just hands you a different tool, cleanly, the moment the question genuinely needs one.

---

# 8. ast-grep: When Regex Stops Being the Right Tool

Regex is excellent for textual navigation.

It becomes less suitable when the question is about actual Rust syntax.

For example:

> Find every call to `unwrap()`, regardless of formatting.

Regex can search for:

```text
unwrap\(
```

but ast-grep can ask a syntax-aware question.

Install ast-grep with:

```bash
cargo install ast-grep --locked
```

The official installation documentation confirms this method. ([Ast Grep][5])

## Structural overview

Current ast-grep provides:

```bash
ast-grep outline src/main.rs
```

For a directory:

```bash
ast-grep outline src/
```

You can select particular top-level item types:

```bash
ast-grep outline src/ --type struct,enum,function
```

You can also request exported items:

```bash
ast-grep outline src/ --items exports
```

`outline` is syntax-aware and provides a compact structural view, but it does not resolve references, types, or call graphs. ([Ast Grep][3])

## Pattern matching

A single AST node can be represented with `$NAME`:

```bash
ast-grep -l rs -p '$OBJECT.unwrap()'
```

Multiple nodes can be represented with `$$$`:

```bash
ast-grep -l rs -p 'process($$$ARGS)'
```

The shell should receive the pattern inside single quotes so that `$OBJECT` and `$$$ARGS` are passed literally to ast-grep.

## Structural rules

For more complex analysis, ast-grep supports YAML rules.

A valid basic rule has separate YAML fields:

```yaml
id: find-unsafe-blocks
language: Rust
rule:
  pattern: unsafe { $$$BODY }
```

Run it with:

```bash
ast-grep scan -r find-unsafe.yaml
```

The current ast-grep CLI supports `scan -r RULE_FILE`, and its rule format uses fields such as `id`, `language`, and `rule`. ([Ast Grep][6])

### Practical boundary

Use:

```text
rg
```

when the question is:

> What text appears here?

Use:

```text
ast-grep
```

when the question is:

> What syntactic structure appears here?

Use:

```text
rust-analyzer
```

when the question is:

> What symbol, type, implementation, or reference does this code actually mean?

---

> **End of Skill Building (ii).** Three tools, three exact boundaries between them, stated as plainly as the appendix ever states anything. Skill Building (iii) moves back into Helix and ripgrep specifically — not new tools this time, just the fine print apprentices are expected to stop confusing between them.
