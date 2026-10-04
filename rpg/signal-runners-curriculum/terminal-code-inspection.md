# Terminal Code Inspection and Navigation

A BVI Rust programmer can treat the terminal as a code-navigation layer in front of Helix.

The basic division is:

* `rg` — find text.
* `ast-grep` — find structure.
* `Helix` — read, navigate, select, and edit.
* `rust-analyzer` (via Helix's LSP integration) — understand semantic relationships when syntax alone is not enough.

The goal is not to dump entire source files into the terminal.

The goal is to progressively narrow the codebase until only the relevant Rust code remains.

---

# 1. The Inspection Pipeline

A useful mental model is a sequence of questions:

- Start with the **CODEBASE**.
- Use `rg` to ask: "Where is it?"
- Use `ast-grep` to ask: "What structure is involved?"
- Use Helix to say: "Let me read it."
- Use textobjects / Tree-sitter to say: "Let me navigate inside it."
- Use Helix's LSP integration (rust-analyzer) to ask: "What does this actually mean — where else does it apply, and what calls it?"

In practice, the tools map to actions:

- `rg` locates.
- `ast-grep` identifies structure.
- Helix inspects.
- textobject isolates.
- Tree-sitter navigation moves.
- rust-analyzer (through Helix's LSP client) understands relationships — definitions, references, types — that are invisible to text and syntax alone.

This keeps each tool doing the job it is best suited for.

---

# 2. ripgrep: Find the Text

`rg` is the first tool to reach for when you know something textual about the code.

Examples:

```bash
rg 'my_function'
```

Find every occurrence of a name.

```bash
rg 'fn my_function'
```

Find occurrences that look like a Rust function declaration.

```bash
rg 'pub fn'
```

Find public function declarations.

```bash
rg '^use\s+'
```

Find `use` declarations beginning at the start of a line.

```bash
rg 'Todo'
```

Find the word `Todo` throughout the repository.

For Rust only:

```bash
rg -t rust 'Todo'
```

This restricts the search to files recognized as Rust.

`rg` automatically respects common ignore rules such as `.gitignore`, which is one reason it works well for repository-wide discovery.

---

# 3. Add Context Instead of Reading the Whole File

Once `rg` finds something interesting, add a small amount of surrounding source.

```bash
rg -A 15 'pub fn process_data'
```

`-A 15` means: show the match plus 15 lines after the match.

Similarly:

```bash
rg -B 5 'pub fn process_data'
```

shows five lines before the match.

And:

```bash
rg -C 5 'pub fn process_data'
```

shows five lines on both sides.

Use context as a **preview**, not as a substitute for structural inspection.

For example:

```bash
rg -n -C 5 'fn process_data'
```

The `-n` option adds line numbers.

This gives you enough information to decide whether the match is the code you want.

`rg` also supports `-U` (`--multiline`), which lets a pattern span multiple lines instead of matching within a single line only. This is useful when the text you're looking for — for example a multi-line macro invocation or an attribute followed by its item — is split across lines that a normal single-line match would miss:

```bash
rg -U 'fn\s+\w+\s*\([^)]*\)\s*\{' src/parser.rs
```
This looks for a function signature and its opening brace even if the signature itself wraps across several lines.

Find multiple match for double-quote `"`.

```
$ rg -U "\"" hello-world/src/main.rs 
12:#[page("/")]
18:                <title>"Hello world"</title>
22:                hello(name: "World")
31:        <h1>"Hello, " (name) "!"</h1>
```

---

# 4. Search for Definitions

For Rust code archaeology, definitions are particularly useful.

```bash
rg -n '^(pub\s+)?fn\s+'
```

Find function definitions.

```bash
rg -n '^(pub\s+)?struct\s+'
```

Find structs.

```bash
rg -n '^(pub\s+)?enum\s+'
```

Find enums.

```bash
rg -n '^(pub\s+)?trait\s+'
```

Find traits.

```bash
rg -n '^(pub\s+)?type\s+'
```

Find type aliases.

```bash
rg -n '^impl'
```

Find implementation blocks.

These are textual patterns. They are useful for rapid discovery, but they do not understand Rust syntax.

For example, a regular expression cannot reliably determine where a Rust function ends when the function contains nested blocks.

That is where `ast-grep` becomes useful.

---

# 5. Search for Imports and Dependencies

Imports are one of the fastest ways to discover what a module depends on.

```bash
rg -n '^use\s+'
```

Find Rust `use` declarations.

Search for a particular crate:

```bash
rg -n 'use\s+tokio'
```

Search for a module:

```bash
rg -n 'use\s+crate::parser'
```

Search for references:

```bash
rg -n 'Parser'
```

Search for a dependency declaration:

```bash
rg -n 'tokio' Cargo.toml
```

For a whole workspace:

```bash
rg -n 'tokio' --glob 'Cargo.toml'
```

This is often enough to answer the first code-archaeology question:

> Where does this thing enter the codebase?

Note that `rg` can only tell you where a name is *written*. It cannot tell you where a `use` statement's crate is actually implemented, or which concrete type a generic parameter resolves to — that is a job for rust-analyzer (see Section 12a).

---

# 6. Use `--glob` to Control the Search

You can explicitly restrict the files being searched.

```bash
rg 'pattern' --glob '*.rs'
```

Rust source only.

```bash
rg 'pattern' --glob '*.toml'
```

Cargo configuration only.

```bash
rg 'pattern' --glob '*.md'
```

Markdown documentation only.

You can combine them:

```bash
rg 'pattern' \
  --glob '*.rs' \
  --glob '*.toml'
```

You can also exclude paths:

```bash
rg 'pattern' --glob '!target/**'
```

This is useful when you want to make the inspection scope explicit rather than relying entirely on ignore files.

---

# 7. Get Precise Locations

When a match is interesting, you want a location.

```bash
rg -n 'fn process_data'
```

Typical output:

```text
src/parser.rs:42:pub fn process_data(...)
```

For machine-friendly `file:line:column` output:

```bash
rg --vimgrep 'fn process_data'
```

This is useful when another tool or workflow needs an exact location.

For example:

```text
src/parser.rs:42:8:pub fn process_data(...)
```

The important information is the file, line, and column.

That turns a repository-wide search result into a navigation target.

Recap of `rg fn` vs `rg --vimgrep fn`:

```
$ rg fn
hello-world/src/main.rs
8:async fn main() {
13:async fn home() -> Result {
29:async fn hello(name: &str) -> Result {

hello-world/Cargo.lock
352:name = "fnv"
487: "fnv",

$ rg --vimgrep fn
hello-world/src/main.rs:8:7:async fn main() {
hello-world/src/main.rs:13:7:async fn home() -> Result {
hello-world/src/main.rs:29:7:async fn hello(name: &str) -> Result {
hello-world/Cargo.lock:352:9:name = "fnv"
hello-world/Cargo.lock:487:3: "fnv",
```

Note the `fnv` hits in `Cargo.lock` — a plain, unanchored search for `fn` matches substrings anywhere, including inside unrelated crate names. This is a good illustration of why textual search alone can produce noise that a structural or semantic tool would filter out.

---

# 8. Helix: Open the Target

Once you have the file and line:

```bash
hx src/parser.rs:42
```

You can then inspect the surrounding source in Helix.

The workflow is: use `rg` to find the location, then open the file at that line with `hx`. For example, `rg` gives you `src/parser.rs:42`, then you run `hx src/parser.rs:42`.

This is much more efficient than opening a large file and manually searching through it.

If you already know the symbol:

```bash
hx src/parser.rs
```

Then inside Helix:

```text
/ process_data
```

Search for the symbol.

Press:

```text
n
```

for the next match.

Press:

```text
N
```

for the previous match.

Helix's `/` search uses regular expressions.

---

# 9. Helix Regex Search

Use Helix when the search has become local to one file.

Examples:

```text
/parse_expression
```

```text
/fn\s+\w+
```

```text
/Result<
```

```text
/unsafe\s*\{
```

The progression is: use `rg` for repository-wide search, then use Helix's `/` for file-level regex search, then use `n` and `N` to move between matches.

This prevents the terminal from becoming a giant source-code dump.

---

# 10. Helix Textobjects: Read Structure

Helix adds another level of navigation.

A **textobject** is a structured piece of text that Helix can select and operate on.

Instead of thinking:

> Select lines 42 through 67.

you can think:

> Select this function.

Or:

> Select this parameter list.

Or:

> Select this comment.

Helix textobjects can refer to ordinary structures such as words and paragraphs, but also syntax-aware programming structures such as functions, types, parameters, comments, and tests. Rust is among the languages with Tree-sitter textobject support.

**Important caveat:** syntax-aware textobjects like `f`, `t`, and `c` require a Tree-sitter grammar to be active for the current file *and* a language-specific `textobjects.scm` query file to be implemented for that grammar. Rust has this support, so the examples below work as written — but if you apply this same workflow to a less-common language, check that the query file exists before relying on these keys.

The basic pattern is:

```text
mi + object
```

Select **inside** the object.

```text
ma + object
```

Select **around** the object.

For Rust inspection, the important objects include:

```text
f    function
t    type / class
a    argument / parameter
c    comment
T    test
```

So:

```text
mif
```

means:

> select inside the current function.

And:

```text
maf
```

means:

> select around the current function.

This is especially useful for a blind programmer because the selection represents a meaningful syntactic region rather than an arbitrary visual rectangle.

---

# 11. Function Navigation

Helix also provides Tree-sitter-aware navigation.

For example:

```text
]f
```

moves to the next function.

```text
[f
```

moves to the previous function.

Likewise, type navigation can use:

```text
]t
```

and

```text
[t
```

These commands operate on syntax-aware structures rather than merely searching for words.

That gives you two complementary forms of movement:

- Regex search: "Find text matching this pattern."
- Tree-sitter navigation: "Move to the next structure of this kind."

---

# 12. ast-grep: Find Structure

`ast-grep` is the next step when textual searching is no longer enough.

It parses source code and searches its syntax tree.

The basic command is:

```bash
ast-grep -p 'PATTERN'
```

`-p` means `--pattern`.

The pattern is written as code.

For example:

```bash
ast-grep -p '$OBJECT.unwrap()'
```

This searches for calls structurally resembling:

```rust
something.unwrap()
```

regardless of the exact variable name.

A `$NAME` metavariable represents one AST node.

A `$$$` metavariable can match multiple AST nodes.

When no path is given, `ast-grep` searches the current directory (`.`) by default and infers the language of each file from its extension, so you generally don't need `--lang` unless you're piping code in via `--stdin`.

---

# 12a. rust-analyzer: Understand Semantic Relationships

`rg` and `ast-grep` both work on text and syntax. Neither can tell you what a piece of code actually *means*: which concrete type a generic resolves to, where a trait method is actually implemented for a given type, or every call site of a function across module and crate boundaries. That requires semantic analysis, which is what rust-analyzer provides through Helix's built-in LSP client.

Once a file is open in Helix and rust-analyzer is running (check with `hx --health rust`), the following keys turn Helix into a semantic navigation tool:

```text
gd     goto definition
gD     goto declaration
gy     goto type definition
gr     goto references (list every call site / usage)
<space>k   hover documentation for the symbol under the cursor
<space>a   list available code actions
<space>r   rename symbol (renames every reference, project-wide)
<space>s   workspace symbol picker
```

This closes the loop with `rg` and `ast-grep`:

- `rg` answers *where does this text occur?*
- `ast-grep` answers *where does this shape occur?*
- rust-analyzer (`gd`, `gr`, `gy`) answers *what does this resolve to, and who actually depends on it?*

A concrete example of when this matters: `rg -n 'Parser'` will find every line that contains the word `Parser`, including comments, unrelated types, and string literals. `gr` on the `Parser` struct's name, by contrast, lists only the places the compiler considers genuine references to that specific symbol — filtering out coincidental textual matches and resolving trait/generic indirection that neither `rg` nor `ast-grep` can see.

---

# 13. Find Function Structures

A function pattern can describe an entire Rust function:

```bash
ast-grep -p 'fn $NAME($$$ARGS) { $$$BODY }'
```

The important distinction is that this is **structural matching**.

`$$$BODY` can represent multiple statements, including nested syntax.

You are no longer trying to make a regular expression understand balanced braces.

The pattern describes a Rust function to the parser, and ast-grep matches the resulting syntax structure.

For public functions:

```bash
ast-grep -p 'pub fn $NAME($$$ARGS) { $$$BODY }'
```

For a particular function:

```bash
ast-grep -p 'fn process_data($$$ARGS) { $$$BODY }'
```

If the function has a return type, include it in the pattern:

```bash
ast-grep -p 'fn $NAME($$$ARGS) -> Result<$TYPE, $$$ERR> { $$$BODY }'
```

The exact pattern needs to match valid Rust syntax.

---

# 14. Find Structs and Enums

For example:

```bash
ast-grep -p 'struct $NAME { $$$FIELDS }'
```

or:

```bash
ast-grep -p 'pub struct $NAME { $$$FIELDS }'
```

Enums:

```bash
ast-grep -p 'enum $NAME { $$$VARIANTS }'
```

Traits:

```bash
ast-grep -p 'trait $NAME { $$$BODY }'
```

Implementation blocks:

```bash
ast-grep -p 'impl $TYPE { $$$BODY }'
```

The point is not memorizing every possible pattern.

The point is learning the progression from text to regex to AST pattern.

Use the simplest representation that answers the question.

---

# 15. Search for Calls

One of the most useful structural searches is finding a particular method call.

```bash
ast-grep -p '$OBJECT.unwrap()'
```

Find calls structurally equivalent to:

```rust
value.unwrap()
result.unwrap()
foo.unwrap()
```

You can search for a method:

```bash
ast-grep -p '$OBJECT.expect($MESSAGE)'
```

Or a function:

```bash
ast-grep -p 'println!($$$ARGS)'
```

The structural match is more robust than searching for the literal text alone.

---

# 16. Search for a Specific Return Shape

Suppose you want functions returning `Result`.

A structural pattern can express that:

```bash
ast-grep -p 'fn $NAME($$$ARGS) -> Result<$TYPE, $$$ERR> { $$$BODY }'
```

This changes the question from:

> Which lines contain `Result`?

to:

> Which functions have a `Result` return type?

That is a structural question.

---

# 17. ast-grep Outline: The Navigation Tool

`ast-grep` provides an especially useful command for code inspection:

```bash
ast-grep outline
```

**Note:** as of this writing, `outline` is an alpha-preview feature (introduced in ast-grep 0.44.0). It's intended for real use, but flags, output format, and language coverage may still change — check `ast-grep outline --help` against your installed version before relying on exact flag names.

`outline` explores code structure rather than dumping source.

It can expose:

* symbols
* imports
* exports
* members
* function signatures
* struct members
* other top-level items

For example:

```bash
ast-grep outline src/parser.rs
```

For a compact structural view:

```bash
ast-grep outline src/parser.rs --view digest
```

For signatures:

```bash
ast-grep outline src/parser.rs --view signatures
```

For a more expanded view:

```bash
ast-grep outline src/parser.rs --view expanded
```

For a directory, the default view groups top-level item names by symbol type, oriented toward discovering the structure of the codebase rather than reading every source line.

This makes `outline` particularly valuable for a terminal-first, screen-reader-first workflow.

Instead of:

```bash
cat src/parser.rs
```

you can begin with:

```bash
ast-grep outline src/parser.rs
```

Then decide which symbol deserves deeper inspection.

---

# 18. A Better Code-Archaeology Sequence

When entering an unfamiliar Rust module:

### Step 1 — Get the shape

```bash
ast-grep outline src/parser.rs
```

You learn what the file contains.

### Step 2 — Find the symbol

```bash
rg -n 'Parser'
```

You locate textual references.

### Step 3 — Find the structure

```bash
ast-grep -p 'struct Parser { $$$FIELDS }' src/parser.rs
```

You confirm the actual struct.

### Step 4 — Open the source

```bash
hx src/parser.rs:LINE
```

### Step 5 — Navigate structurally

Inside Helix:

```text
]f
[f
]t
[t
```

### Step 6 — Select the relevant structure

```text
mif
maf
mit
mat
```

### Step 7 — Confirm the semantics

With the cursor on the symbol, use rust-analyzer through Helix:

```text
gr     see every real reference, project-wide
gd     jump to where it's actually defined
<space>k   read its documentation without leaving the file
```

### Step 8 — Read the surrounding code

Now the source is small enough to understand, and you know not just what it looks like but what depends on it.

---

# 19. Terminal Inspection Should Progress From Broad to Narrow

A good inspection session moves from repository to file to symbol to structure to function to expression to specific line — and, when needed, from syntax to semantics.

The tools follow the same progression: `rg`, then `ast-grep outline`, then `ast-grep` pattern, then Helix `/`, then Helix textobject, then Tree-sitter navigation, then rust-analyzer (`gd`/`gr`) when the question can no longer be answered by syntax alone.

Each step reduces uncertainty.

---

# 20. Four Questions, Four Tools

When inspecting code, ask which question you are actually asking.

### "Where does this text occur?"

Use:

```bash
rg
```

Example:

```bash
rg -n 'process_data'
```

### "Where does this structure occur?"

Use:

```bash
ast-grep
```

Example:

```bash
ast-grep -p '$OBJECT.unwrap()'
```

### "How do I move through this source?"

Use:

```text
Helix
```

Examples:

```text
/...
n
N
]f
[f
]t
[t
mif
maf
```

### "What does this actually resolve to, and who depends on it?"

Use Helix's LSP integration with rust-analyzer:

```text
gd
gr
gy
<space>k
```

---

# 21. Read-Only Discipline

For code inspection, keep the tools in their read-only roles.

With `rg`:

```bash
rg ...
```

With ast-grep:

```bash
ast-grep -p '...'
ast-grep outline ...
```

Avoid `--rewrite` while performing reconnaissance.

ast-grep does support rewriting, but that is a separate operation. The `run` command handles both search and rewrite, so the inspection workflow deliberately uses only its search capabilities.

Likewise, `gr` (goto references) and `gd` (goto definition) are read-only navigation. Reserve `<space>r` (rename symbol) and `<space>a` (code actions) for when you've deliberately moved from inspection into editing.

The mental separation is:

- DISCOVER with `rg`
- UNDERSTAND STRUCTURE with `ast-grep`
- UNDERSTAND MEANING with rust-analyzer (`gd`, `gr`, `gy`, `<space>k`)
- READ with Helix
- EDIT with Helix

---

# 22. The Core Terminal Toolkit

For your workflow, the essential commands are small.

## ripgrep

```bash
rg 'text'
rg -n 'text'
rg -C 5 'text'
rg -A 15 'text'
rg -U 'text'
rg -t rust 'text'
rg --glob '*.rs' 'text'
rg --vimgrep 'text'
```

## ast-grep

```bash
ast-grep outline src/
ast-grep outline src/file.rs --view signatures
ast-grep -p 'PATTERN'
ast-grep -p 'PATTERN' src/
ast-grep -p 'PATTERN' -l rust
```

## Helix

```bash
hx src/file.rs
hx src/file.rs:42
```

Then, for text and structure:

```text
/regex
n
N
[f
]f
[t
]t
mif
maf
mit
mat
```

And, for semantics (rust-analyzer via LSP):

```text
gd
gD
gy
gr
<space>k
<space>a
<space>r
<space>s
```

---

# 23. ast-grep metavariables

In ast-grep, the "special values" are not a fixed set of reserved names like `$OBJECT` or `$MESSAGE`. Instead, they are syntactic forms you use to create metavariables. The names themselves, such as `OBJECT`, `MESSAGE`, or `ARGS`, are arbitrary. You choose them.

## Meta Variable Forms

### `$VAR`

Named single capture.

Matches exactly one AST node and stores it under the name `VAR`.

Example:

```bash
$OBJECT.unwrap()
```

### `$_`

Dropped single capture.

Matches one AST node without capturing it. Useful when you need to match a node but do not need its value. Names starting with `_` are also anonymous, meaning repeated occurrences of `$_` in the same pattern are not required to match the same content.

### `$_123`

Dropped single capture with a suffix.

Same as `$_`, but with a numeric suffix. The name is ignored.

### `$$$`

Unnamed multi-capture.

Matches zero or more AST nodes.

Example:

```bash
console.log($$$)
```

### `$$$VAR`

Named multi-capture.

Matches zero or more AST nodes and stores them as a sequence under `VAR`.

Example:

```bash
console.log($$$ARGS)
```

### `$$OP`

Unnamed-node capture.

By default, a `$VAR`-style metavariable only captures *named* AST nodes. Matches unnamed nodes such as operators and punctuation. This is a special form for non-identifier tokens.

Example:

```bash
$A $$OP $B
```

## Naming Rules

Meta variable names must follow these rules:

- Start with `$` for single captures, or `$$$` for multi-captures.
- The name is composed of uppercase letters `A` through `Z`, underscore `_`, or digits `1` through `9`.
- The name must be the whole content of an AST node.

Partial text does not work. For example, `prefix$VAR`, `"hello $NAME"`, or `a $OP b` will not work. You must match the entire node. Use `$$OP` when you need to match an operator.

Valid names include:

- `$META`
- `$META_VAR`
- `$META_VAR1`
- `$_`
- `$_123`

Invalid names include:

- `$invalid`
- `$Svalue`
- `$123`
- `$KEBAB-CASE`
- `$`

## Notes on Special Names

There are no reserved names like `$MATCH` or `$OBJECT`. Any valid identifier you choose becomes a metavariable.

The same metavariable name can be reused in a pattern to enforce equality. For example, `$A == $A` will only match `x == x`, not `x == y`. This equality check does not apply to anonymous (`$_`-prefixed) metavariables — see the `$_` note above.

Multi-captures such as `$$$VAR` are stored as a list of nodes. Single captures such as `$VAR` store one node.

## Example Patterns

```bash
# Single capture: matches any receiver calling .unwrap()
ast-grep -p '$OBJECT.unwrap()'

# Multi-capture: matches any arguments to console.log
ast-grep -p 'console.log($$$ARGS)'

# Named multi-capture: matches any function body and arguments
ast-grep -p 'fn $NAME($$$ARGS) { $$$BODY }'

# Dropped capture: matches any single argument without storing it
ast-grep -p 'foo($_)'

# Unnamed-node capture: matches a binary operator
ast-grep -p '$A $$OP $B'
```

The key distinction is between single captures, written with `$`, and multi-captures, written with `$$$`. The names you assign are entirely up to you. They simply become labels for the captured nodes.

# 24. The Practical Mental Model

Think of the tools as four different kinds of questions.

`rg` asks:

```text
WHERE IS THIS TEXT?
```

`ast-grep` asks:

```text
WHERE IS THIS CODE SHAPE?
```

Helix asks:

```text
HOW DO I MOVE THROUGH AND READ THIS CODE?
```

rust-analyzer, through Helix's LSP client, asks:

```text
WHAT DOES THIS ACTUALLY MEAN, AND WHO DEPENDS ON IT?
```

And Tree-sitter gives Helix structural awareness of functions, types, parameters, comments, and tests — while rust-analyzer gives it semantic awareness of definitions, references, and types.

This is much more powerful than treating a Rust repository as a collection of flat text files.

The terminal becomes the **discovery layer**.

Helix becomes the **inspection and navigation layer**, and, via rust-analyzer, the **semantic layer**.

The editor does not need to be responsible for discovering the whole repository, and the terminal does not need to reproduce the entire source file.

The workflow is: DISCOVER, then LOCATE, then STRUCTURE, then OPEN, then NAVIGATE, then CONFIRM MEANING, then UNDERSTAND.

That is the core of terminal-first Rust code archaeology.

- `rg` answers *where*, AST patterns answer *what shape*, rust-analyzer answers *what it means and who depends on it*.
- ast-grep `outline` gives you a compact **map of the code before you start reading it**.

---

# Sources

- ripgrep flags: standard `rg --help` reference (`-A`, `-B`, `-C`, `-n`, `-t`, `--glob`, `--vimgrep`, `-U`/`--multiline`).
- Helix textobjects and keymap: [Helix documentation — Textobjects](https://docs.helix-editor.com/textobjects.html), [Helix documentation — Adding textobject queries](https://docs.helix-editor.com/guides/textobject.html), [Helix documentation — Language servers](https://docs.helix-editor.com/master/lsp.html).
- ast-grep pattern syntax and metavariables: [ast-grep — Pattern Syntax](https://ast-grep.github.io/guide/pattern-syntax.html), [ast-grep — Pattern Parse](https://ast-grep.github.io/advanced/pattern-parse.html).
- ast-grep `outline` command: [ast-grep — outline CLI reference](https://ast-grep.github.io/reference/cli/outline.html), [ast-grep — Introducing Outline](https://ast-grep.github.io/blog/ast-grep-outline.html) (alpha preview since v0.44.0).
- ast-grep `run` defaults (path/language inference): [ast-grep — run CLI reference](https://ast-grep.github.io/reference/cli/run.html).
