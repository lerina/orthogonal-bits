# THE DESCENT DRILLS — Skill Building (III)

*Continued from Skill Building (ii). The tool boundaries are drawn. Now the appendix comes back down to earth — the actual keystrokes, in the actual two tools apprentices use every day.*

---

> Nothing new gets introduced in this installment, and instructors say so up front, almost as a relief after the last one. Helix and ripgrep have both been named a dozen times already in this curriculum. This is just the appendix finally sitting still long enough to spell out, command by command, what each one actually expects you to type — and, more importantly, which of those commands quietly do not exist in the other tool no matter how similar the two feel.

---

# 9. Regex Inside Helix

Helix uses `/` for forward regex search:

```text
/pattern
```

Reverse search:

```text
?pattern
```

Next match:

```text
n
```

Previous match:

```text
N
```

These are current Helix search commands. ([Helix][7])

Example:

```text
/^\s*fn\s+\w+
```

This searches the current document for likely Rust function declarations.

## Regex replacement in Helix

Helix does not use a Vim-style substitution command such as:

```text
:%s/foo/bar/g
```

Instead, its normal regex replacement workflow is based on selections.

For example:

```text
%
s
foo
Enter
c
bar
Esc
```

Conceptually:

1. `%` selects the whole buffer.
2. `s` selects all regex matches inside the selection.
3. `c` changes the selected matches.
4. Type the replacement.
5. `Esc` returns to normal mode.

Helix documents this workflow explicitly. ([Helix][2])

The important distinction is:

> **Helix searches with regex, then operates on the resulting selections.**

---

> **Drill note:** apprentices coming from other editors reliably try the Vim-style substitution command at least once before this section talks them out of it. Instructors let that happen too, same as the file-browser reflex back in Part One. The correction lands harder once the wrong muscle memory has already failed once, in front of everyone, on a live terminal.

---

# 10. Regex Inside ripgrep

The simplest ripgrep search is:

```bash
rg 'pattern'
```

ripgrep searches recursively from the current directory and follows its normal file-selection and ignore rules.

Useful options include:

```bash
rg -F 'pattern'
```

Literal fixed-string search.

```bash
rg -i 'error'
```

Case-insensitive search.

```bash
rg -w 'fn'
```

Whole-word search.

```bash
rg -l 'pattern'
```

Print only filenames containing matches.

```bash
rg -c 'pattern'
```

Count matching lines per file.

```bash
rg -A 2 'pattern'
```

Show two lines after each match.

```bash
rg -B 2 'pattern'
```

Show two lines before each match.

```bash
rg -C 2 'pattern'
```

Show two lines before and after each match.

These are standard ripgrep facilities. ([Docs.rs][8])

## Quote your regex

Prefer:

```bash
rg '^\s*(pub\s+)?fn\s+\w+'
```

rather than:

```bash
rg ^\s*(pub\s+)?fn\s+\w+
```

Quoting prevents the shell from interpreting characters before ripgrep receives the pattern.

### The rule

> **Quote the pattern so the regex engine, not the shell, interprets it.**

---

# 11. Common ripgrep Searches for Rust

Find likely function definitions:

```bash
rg '^\s*(pub\s+)?(async\s+)?fn\s+\w+' -g '*.rs'
```

Find `println!` calls:

```bash
rg 'println!\(' -g '*.rs'
```

Find TODO markers:

```bash
rg 'TODO' -g '*.rs'
```

Find error or fatal log lines:

```bash
rg '^(ERROR|FATAL)' -g '*.rs'
```

Find imports from a particular crate:

```bash
rg 'use\s+some_crate::' -g '*.rs'
```

Find likely functions returning `Result`:

```bash
rg 'fn\s+\w+[^{}]*->\s*Result' -g '*.rs'
```

These are **textual searches**, not guarantees about Rust syntax.

For example:

```bash
rg 'fn\s+\w+[^{}]*->\s*Result'
```

may produce false positives or miss valid Rust declarations.

That is acceptable when the purpose is reconnaissance.

### The rule

> **Use regex to generate useful leads. Verify important conclusions with the compiler, editor, tests, or a syntax-aware tool.**

---

> **End of Skill Building (iii).** Two tools, spelled out completely, side by side, with the unquoted-pattern mistake and the Vim-substitution reflex both flagged as the two errors nearly every apprentice makes at least once. Skill Building (iv) is where the appendix finally puts Helix and ripgrep next to each other on purpose, to make sure nobody leaves this curriculum still mixing the two up.
