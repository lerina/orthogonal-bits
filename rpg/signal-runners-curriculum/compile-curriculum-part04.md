# Read the Relevant Rust Code in Helix

## 1. Helix Is Selection-First

Helix follows a selection-first editing model.

The basic idea is:

> **Select the thing you want to operate on, then perform the operation.**

A selection can be:

* a word
* a line
* several lines
* a regex match
* a function
* a type
* a parameter
* a comment
* a test
* a surrounding pair
* another syntax-aware textobject

This is particularly useful when reading Rust because you can select a meaningful piece of code rather than manually estimating its beginning and end.

---

# 2. Textobjects

In Helix, a **textobject** is a structured piece of text that you can select, manipulate, or operate on.

Textobjects can represent ordinary textual structures such as:

```text
word
WORD
paragraph
parentheses
brackets
quotes
```

They can also represent programming-language structures such as:

```text
function
type
parameter
comment
test
```

Helix's documentation defines textobjects as a way to refer to pieces of text according to their structure or purpose. ([Helix][1])

This is especially important for Rust.

Instead of thinking:

> "The function starts around here and ends somewhere down there."

you can think:

> "Select the function."

That is a much better abstraction for code navigation.

---

# 3. Selecting a Textobject

Helix uses:

```text
mi
```

to select **inside** a textobject.

And:

```text
ma
```

to select **around** a textobject.

The distinction is similar to:

```text
mi = inside
ma = around
```

For example:

```text
mif
```

selects inside a function.

```text
maf
```

selects around a function.

For a type:

```text
mit
mat
```

For a parameter:

```text
mia
maa
```

For a comment:

```text
mic
mac
```

For a test:

```text
miT
maT
```

These language-aware textobjects depend on Tree-sitter support and the appropriate textobject query for the language. ([Helix][1])

---

# 4. The Core Rust Textobjects

The important Helix textobjects for Rust code reading are:

```text
f   function
t   type
a   argument / parameter
c   comment
T   test
```

So:

```text
mif
```

means:

> Select inside the current function.

```text
maf
```

means:

> Select around the current function.

Likewise:

```text
mit
```

selects inside a type.

```text
mat
```

selects around a type.

```text
mia
```

selects inside a parameter.

```text
mic
```

selects inside a comment.

These are not regex searches.

They are **syntax-aware selections** based on Tree-sitter.

Helix currently lists Rust as having Tree-sitter textobject support. ([Helix][2])

---

# 5. Textobjects Versus Regex

This distinction is important.

Regex asks:

> **Does this text have a particular shape?**

For example:

```text
^\s*(pub\s+)?(async\s+)?fn\s+\w+
```

This finds text that looks like a Rust function declaration.

It does not actually understand the Rust function.

A textobject asks:

> **Where is the function according to the syntax tree?**

That makes the two tools complementary.

Use regex when you want to **find** something.

Use a textobject when you want to **select the structure around it**.

The workflow becomes:

```text
rg
 ↓
find the file
 ↓
Helix
 ↓
search
 ↓
textobject
 ↓
read the relevant code
```

---

# 6. Search With Regex, Then Select the Function

Suppose `rg` tells you that the function you want is in:

```text
src/parser.rs
```

Open it:

```bash
hx src/parser.rs
```

Inside Helix, search:

```text
/parse_expression
```

Press:

```text
n
```

to move to the next match.

Once the cursor is inside the function, use:

```text
mif
```

Now the function body is selected.

If you want the complete function structure including its surrounding declaration, use:

```text
maf
```

This creates a very useful separation:

```text
Regex search
    ↓
locate symbol
    ↓
Tree-sitter textobject
    ↓
select structure
```

---

# 7. Read the Function as a Unit

Once the function is selected, the selection itself becomes a cognitive landmark.

You know:

> This is one function.

You no longer need to remember its visual boundaries.

You can then inspect the selection, copy it, format it, replace it, or otherwise operate on it.

For code understanding, the important operation is often simply:

```text
mif
```

followed by reading the selected region.

The point is not necessarily to modify the code.

The point is to establish:

```text
FUNCTION
    ↓
parameters
    ↓
body
    ↓
calls
    ↓
return value
```

---

# 8. Move Between Functions Without Searching

Helix also supports Tree-sitter-based navigation between textobjects.

For example:

```text
]f
```

moves to the next function.

```text
[f
```

moves to the previous function.

Similarly:

```text
]t
```

moves to the next type.

```text
[t
```

moves to the previous type.

This is different from regex searching.

Regex says:

> Find text matching this pattern.

Tree-sitter navigation says:

> Move to the next syntactic object of this kind.

Helix documents this as Tree-sitter textobject navigation. ([Helix][1])

---

# 9. Textobjects as Code Landmarks

For auditory code reading, textobjects provide useful structural landmarks.

Instead of hearing an uninterrupted stream of source code, you can establish:

```text
function
    parameter
    body
        comment
        expression
        call
        branch
        return
```

The textobject gives you a known boundary.

This makes it easier to answer:

> Where am I?

without relying on visual layout.

For example:

```text
mif
```

means:

> I am examining this function as a unit.

Then:

```text
mia
```

can focus on a parameter.

And:

```text
mic
```

can isolate a comment.

The important idea is:

> **Textobjects turn source code into navigable structural units.**

---

# 10. Regex and Textobjects Work Together

A practical Rust investigation can therefore use two different kinds of queries.

### Text query

Use regex:

```text
/parse_expression
```

Question:

> Where is this text?

### Structural query

Use a textobject:

```text
mif
```

Question:

> What is the function containing this location?

### Structural navigation

Use:

```text
]f
```

Question:

> What is the next function?

### Repository query

Use `rg`:

```bash
rg -n 'parse_expression' -g '*.rs'
```

Question:

> Where does this symbol appear throughout the repository?

This gives a useful division of labor:

```text
rg
repository-wide text search

Helix regex
file-level text search

Helix textobjects
syntax-aware selection

Helix Tree-sitter navigation
syntax-aware movement

rust-analyzer
semantic relationships
```

---

# 11. Textobjects and Selection Operations

Because Helix is selection-first, a textobject selection can immediately become the target of another operation.

For example:

```text
mif
```

selects inside a function.

Then:

```text
y
```

yanks the selection.

Or:

```text
c
```

changes the selection.

Or:

```text
d
```

deletes it.

Or:

```text
=
```

formats the selection.

The important concept is:

```text
SELECT then    ACT
```

rather than:

```text
ACT then    figure out what the command should operate on
```

Helix's documentation explicitly describes this selection-first model. ([Helix][3])

---

# 12. Regex Can Create Selections Too

Textobjects are not the only way to create structured selections.

Helix also has regex-based selection commands.

For example:

```text
%
```

selects the entire document.

Then:

```text
s
```

selects all regex matches inside the current selection.

So:

```text
%
s
TODO
Enter
```

can select all `TODO` matches in the document.

Helix also supports:

```text
S
```

to split selections on regex matches.

These commands are part of Helix's selection system. ([Helix][4])

This gives you another useful distinction:

```text
regex selection
    = select text matching a pattern

textobject selection
    = select text matching a syntactic structure
```

---

# 13. A Practical Rust Reading Loop

For a new Rust repository:

### Step 1 — Find the symbol

From the terminal:

```bash
rg -n 'parse_expression' -g '*.rs'
```

### Step 2 — Open the relevant file

```bash
hx src/parser.rs
```

### Step 3 — Search inside the file

```text
/parse_expression
```

### Step 4 — Move through matches

```text
n
```

### Step 5 — Select the containing function

```text
mif
```

### Step 6 — Read the function

Identify:

```text
parameters
local data
branches
calls
returns
errors
```

### Step 7 — Move to related functions

```text
]f
[f
```

### Step 8 — Search for another symbol

```text
/next_symbol
```

This produces a repeatable loop:

```text
RG then    SEARCH then    TEXTOBJECT then    READ then    NAVIGATE
```

---

# 14. The Important Mental Model

There are three different questions.

### "Where is it?"

Use:

```text
rg
```

or:

```text
/
```

### "What structure contains it?"

Use:

```text
mi...
ma...
```

### "What structure comes next?"

Use:

```text
]f
[f
]t
[t
```

This is more useful than treating regex as the universal solution.

Regex is excellent at **finding textual patterns**.

Tree-sitter textobjects are excellent at **finding syntactic structures**.

Rust-analyzer is appropriate when you need **semantic relationships**.

---

# 15. The BVI Reading Principle

For source-code reading, use the smallest abstraction that answers the question.

```text
Literal
  ↓
Regex
  ↓
Textobject
  ↓
Tree-sitter navigation
  ↓
rust-analyzer
```

A literal search answers:

> Where does this exact text occur?

Regex answers:

> Where does text with this shape occur?

A textobject answers:

> What syntactic unit am I inside?

Tree-sitter navigation answers:

> Where is the next syntactic unit of this kind?

rust-analyzer answers:

> What does this symbol mean and what is related to it?

The result is a practical code-reading system:

```text
SEARCH
    ↓
LOCATE
    ↓
SELECT
    ↓
READ
    ↓
TRACE
```

`rg` finds the territory.

Helix regex finds the textual landmark.

Helix textobjects establish the structural boundary.

Tree-sitter navigation moves between structural landmarks.

rust-analyzer provides semantic relationships.

> **Read the relevant Rust code as structure, not as an undifferentiated stream of lines.**

NOTE: **textobjects is not merely another kind of regex.** Helix's function/type/parameter/comment/test textobjects are Tree-sitter-based syntax queries. That makes them particularly valuable for your Rust-reading workflow: `rg` gets you to the neighborhood, while `mif`/`maf` and `]f`/`[f` let Helix establish the actual code structure. ([Helix][1])

[1]: https://docs.helix-editor.com/textobjects.html "Textobjects"
[2]: https://docs.helix-editor.com/lang-support.html "Language support"
[3]: https://docs.helix-editor.com/master/usage.html "Usage"
[4]: https://docs.helix-editor.com/master/commands.html "Commands"


