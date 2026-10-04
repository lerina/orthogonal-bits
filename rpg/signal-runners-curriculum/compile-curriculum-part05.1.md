# THE DESCENT DRILLS — Skill Building (I)

*Continued from Part Four. This part runs long enough that it's broken into installments of its own — as many as it takes. This is the first.*

---

> Skill Building is where the curriculum does something it hasn't done up to now: it stops teaching new drills and folds back in on itself. Apprentices are told, before this section starts, that what follows is lifted straight out of the Field Manual's own regex appendix — the same document that eventually walks a runner through a RUSTCORE checkpoint, here doing the much smaller job of explaining a single character class. Instructors treat this as the moment the whole curriculum finally proves its own claim out loud: the discipline doesn't change shape depending on what it's pointed at. A regex, a function, a vault — same questions, same care, just a different scale of consequence if you get it wrong.

---

# Decomposing Source Code with grep and ripgrep

Text from "The BVI Rust Programmer's Manual"

Debian 13.6, MATE, Orca 48.1

*Regex for Helix and ripgrep*

---

## 1. Regex as a Superpower for Text Navigation

Regular expressions let you search for a *pattern* rather than only a literal string.

For a screen-reader user, this is especially valuable: instead of visually scanning a large file or a long search result, you describe the shape of the text you want and let the tool find it.

Regex is useful for finding things like:

* function definitions
* struct definitions
* trait definitions
* implementations
* error messages
* TODO comments
* Rust syntax
* imports
* types
* repeated textual patterns

Helix and ripgrep both use the Rust regex ecosystem, but they are different tools with different interfaces.

The important distinction is:

> **Regex describes what to match. The tool determines how the search is performed.**

Do not assume that a command or feature from one tool exists in the other.

### The rule

> **Regex turns searching from a visual scan into a logical query.**

---

# 2. Basic Regex Syntax

The following syntax is common to the Rust regex family.

| Pattern  | Meaning                                              | Example                           |
| -------- | ---------------------------------------------------- | ---------------------------------- |
| `.`      | Any character except newline by default              | `a.b` matches `aab`, `acb`, `a1b` |
| `\d`     | Unicode decimal digit                                | `\d{3}` matches `123`             |
| `\w`     | Unicode word character                               | `\w+` matches `hello123`          |
| `\s`     | Unicode whitespace                                   | `\s+` matches spaces and tabs     |
| `^`      | Start of a line in multiline/line-oriented searching | `^fn`                             |
| `$`      | End of a line in multiline/line-oriented searching   | `;$`                              |
| `[abc]`  | One character from the class                         | `[aeiou]`                         |
| `[^0-9]` | One character not in the class                       | anything except an ASCII digit    |
| `(...)`  | Capturing group                                      | `(fn\s+\w+)`                      |
| `\|`     | Alternation (OR)                                     | `error\|warning`                  |

The Rust regex crate defines `\d`, `\s`, and `\w` as Unicode-aware character classes by default. If you specifically want ASCII classes, use forms such as `[0-9]` or `[[:word:]]`. ([Docs.rs][4])

### Important correction

This:

```text
error\|warning
```

does **not** mean "error OR warning" in Rust regex syntax.

Use:

```text
error|warning
```

### The rule

> **Character classes describe characters. Alternation describes alternatives.**

---

# 3. Essential Quantifiers

| Quantifier | Meaning         | Example   |
| ---------- | --------------- | --------- |
| `*`        | Zero or more    | `\d*`     |
| `+`        | One or more     | `\w+`     |
| `?`        | Zero or one     | `colou?r` |
| `{n}`      | Exactly n       | `\d{3}`   |
| `{n,m}`    | Between n and m | `\d{2,4}` |

Examples:

```text
\d*
```

Zero or more digits.

```text
\d+
```

One or more digits.

```text
\w{3}
```

Exactly three Unicode word characters.

```text
\d{2,4}
```

Between two and four Unicode decimal digits.

### The rule

> **Quantifiers control how many times the preceding expression may occur.**

---

# 4. Character Classes and Shorthand

| Pattern | Meaning                      |
| ------- | ---------------------------- |
| `\d`    | Unicode decimal digit        |
| `\D`    | Not a Unicode decimal digit  |
| `\w`    | Unicode word character       |
| `\W`    | Not a Unicode word character |
| `\s`    | Unicode whitespace           |
| `\S`    | Not Unicode whitespace       |
| `\b`    | Word boundary                |
| `\B`    | Not a word boundary          |

A word boundary is a **position**, not a character.

For example:

```text
\bfn\b
```

finds `fn` as a complete word rather than the `fn` inside a longer word.

For ASCII-only matching, use explicit ASCII classes when that is clearer:

```text
[0-9]
[A-Za-z0-9_]
[[:word:]]
```

The Rust regex crate documents these Unicode and ASCII distinctions explicitly. ([Docs.rs][4])

### The rule

> **Character classes describe characters. Boundaries describe positions between characters.**

---

> **Drill note:** four sections in, and instructors point out something apprentices rarely notice on their own — every "rule" callout so far has been a version of the same warning, restated for a different piece of syntax: know exactly what you're describing, and don't let one concept borrow the shape of another. Characters aren't alternatives. Positions aren't characters. The Field Manual's discipline, one character class at a time.
>
> *Continued in Skill Building (ii).*
