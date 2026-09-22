# Rust Debugging with LLDB

## A Progressive Manual for Rust Programmers

### Debian, Cargo, rust-lldb, and LLDB

---

# Preface

## Who this manual is for

This manual is for Rust programmers who already know how to write and build Rust programs but have not yet made LLDB part of their daily workflow.

It is not an LLDB reference manual. It is a guide to using LLDB as an instrument for answering runtime questions about Rust programs.

You do not need to know C, C++, assembly, or debugger internals. You do need patience, curiosity, and a willingness to form hypotheses instead of guessing.

## How to use this manual

Read the early sections in order. They establish the mental model:

```text
Rust source
→ Cargo
→ rustc
→ executable + debug information
→ LLDB
→ observation
→ hypothesis
→ verification
```

After that, use the later sections as recipes. When a bug appears, choose the recipe that matches the shape of the problem:

```text
panic
wrong branch
state corruption
async hang
thread race
FFI boundary
unsafe code
test failure
```

## Accessibility note

Many debugger tutorials assume visual scanning of large dumps. This manual favors a screen-reader-friendly style:

```text
inspect one value at a time
use full command names when they are clearer
use help to discover syntax
prefer small, targeted output
automate repetitive observation with breakpoint commands
```

The goal is not to avoid output. The goal is to make output meaningful.

---

# Prerequisites and Setup on Debian

## Install LLDB

On Debian, install LLDB and a recent Clang toolchain:

```bash
sudo apt update
sudo apt install -y lldb clang
```

Verify:

```bash
lldb --version
```

## Install Rust and Cargo

If Rust is not already installed:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
```

Verify:

```bash
rustc --version
cargo --version
```

## Verify `rust-lldb`

Rust ships a wrapper called `rust-lldb`:

```bash
rust-lldb --version
```

If the command is not found, ensure Cargo’s bin directory is on your `PATH`:

```bash
echo "$PATH"
```

Usually that means:

```bash
export PATH="$HOME/.cargo/bin:$PATH"
```

## Understand debug information

Debug information is the bridge between machine code and Rust source.

Without debug information, LLDB may still run the program, but it cannot reliably answer:

```text
Which Rust function is this?
Which source line is this?
What is this variable?
What type does it have?
```

Cargo’s default debug profile normally includes debug information:

```bash
cargo build
```

The release profile normally does not:

```bash
cargo build --release
```

If you need a release-like build with debug information:

```toml
[profile.release]
debug = true
```

You can also choose a lighter level:

```toml
[profile.release]
debug = "line-tables-only"
```

That gives line numbers without full variable information.

## Confirm the binary

Before launching LLDB, confirm which executable you are debugging:

```bash
ls -l target/debug/
```

Then:

```bash
rust-lldb target/debug/your_program
```

Debugging the wrong binary is a common source of misleading evidence.

---

# 1. What Debugging Actually Is

Debugging is not guessing what the program does.

Debugging is observing what the program actually does.

When a program behaves incorrectly, there are usually several possible explanations.

The debugger lets you replace speculation with evidence.

A useful debugging loop is:

```text
Observe
→ Stop
→ Locate
→ Inspect
→ Trace
→ Form a hypothesis
→ Test the hypothesis
→ Fix
→ Verify
```

LLDB is the instrument used to perform that investigation.

Rust adds another layer:

```text
Rust source
→ Cargo
→ rustc
→ executable + debug information
→ LLDB
```

The debugger sees the compiled program.

Debug information connects that compiled program back to Rust source locations, functions, types, variables, and lines.

---

# Why Rust Debugging Is Different

Rust adds several complications that do not appear in a simple C or C++ debugging session.

## Name mangling

Rust functions are often compiled into symbols with mangled names. A source-level name like:

```rust
parse_config
```

may become something like:

```text
_ZN7my_crate12parse_config17h...
```

LLDB can often match the human name, but not always. This is why source-line breakpoints are sometimes more predictable than function-name breakpoints.

## Optimization

Rust’s optimizer can:

```text
inline functions
reorder instructions
remove variables
merge values
specialize generics
eliminate branches
```

When optimization is aggressive, a variable may be unavailable, a step may jump unexpectedly, or a backtrace may omit frames.

This is not LLDB failing. It is LLDB observing optimized machine code.

## The expression evaluator is not Rust

LLDB’s `expression` command does not use `rustc`. It has some Rust support, but it is not a Rust compiler.

Expressions involving:

```text
iterators
closures
trait methods
generic bounds
async machinery
macros
```

may fail or behave differently from Rust source.

Use `frame variable` for inspection. Use `expr` only when you need it.

## Async Rust

An `async fn` does not execute as a simple synchronous call. It creates a future. An executor polls that future. Execution can suspend at `.await` and resume later.

So an async source call chain is not the same as a synchronous stack trace. LLDB may show runtime frames between your application frames.

## Generics and traits

Generic code can be compiled into multiple specializations. A breakpoint on a generic function may resolve to one or more concrete instantiations.

Trait method calls may dispatch dynamically. The source line may show:

```rust
handler.handle(request)
```

but the runtime may enter a completely different concrete implementation.

## Macros

Macros expand before compilation. A breakpoint on a macro invocation may not correspond to a clean source line. Often it is better to break on the expanded function or the code the macro generates.

## Ownership is static; runtime state is dynamic

Ownership, borrowing, and lifetimes are checked at compile time. LLDB observes runtime memory and values. It cannot explain why the borrow checker rejected your code.

Use the compiler for static rules. Use LLDB for runtime behavior.

---

# 2. The Rust Debugging Stack

A normal Rust debugging session involves several tools.

## Cargo

Cargo builds and runs the project.

```bash
cargo build
cargo run
cargo test
```

For debugging, the important point is that Cargo knows how to build the correct Rust target.

## rustc

`rustc` compiles Rust source code.

The `-g` option is equivalent to:

```text
-C debuginfo=2
```

and enables debug information.

Cargo's normal debug profile provides debug information suitable for ordinary debugging.

## rust-lldb

Rust distributions provide:

```bash
rust-lldb
```

This launches LLDB with Rust-oriented debugger support. Rust has provided `rust-gdb` and `rust-lldb` specifically for this purpose.

For Rust work, prefer:

```bash
rust-lldb
```

over manually launching:

```bash
lldb
```

when the Rust wrapper is available.

## LLDB

LLDB controls the running program.

It can:

```text
start the program
stop execution
set breakpoints
step through instructions
inspect variables
inspect stack frames
inspect threads
evaluate expressions
watch memory changes
continue execution
inspect crashes
```

The Rust compiler produces the program.

LLDB investigates the program.

---

# The Rust/LLDB Boundary

Think of the debugging stack as two worlds:

```text
Rust world
    source code
    types
    ownership
    modules
    async tasks

Machine world
    registers
    memory
    instructions
    threads
    addresses
```

Debug information is the map between them.

LLDB lives in the machine world. It reads debug information to translate:

```text
address → source line
symbol → function name
stack slot → variable
type ID → Rust type
```

When that translation works, debugging feels source-level.

When it breaks, you see the machine world:

```text
?? 
no variable available
optimized out
unknown type
mangled symbol
```

That does not mean the debugger is useless. It means the map is incomplete for that point in the program.

`rust-lldb` is a wrapper that launches LLDB with Rust-oriented support. Prefer it over plain `lldb` when debugging Rust, because it can load Rust-specific pretty-printers and settings.

---

# 3. First Debugging Session

Create or use a small Rust program.

For example:

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

fn main() {
    let x = 10;
    let y = 20;
    let result = add(x, y);

    println!("{result}");
}
```

Build it:

```bash
cargo build
```

Then launch:

```bash
rust-lldb target/debug/your_program
```

You should reach the LLDB prompt:

```text
(lldb)
```

You have now entered the debugger.

---

# A More Complete First Session

Here is a fuller first session with context.

Program:

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

fn main() {
    let x = 10;
    let y = 20;
    let result = add(x, y);

    println!("{result}");
}
```

Build:

```bash
cargo build
```

Launch:

```bash
rust-lldb target/debug/your_program
```

At the LLDB prompt:

```text
(lldb) breakpoint set --name main
Breakpoint 1: where = your_program`main, address = ...
```

Run:

```text
(lldb) run
Process 12345 stopped
* thread #1, name = 'your_program', stop reason = breakpoint 1.1
    frame #0: your_program`main at src/main.rs:6
```

Now inspect the frame:

```text
(lldb) frame info
```

You should see:

```text
frame #0: your_program`main at src/main.rs:6
```

Step to the next line:

```text
(lldb) next
```

Inspect local variables:

```text
(lldb) frame variable
```

You may see:

```text
(int) x = 10
(int) y = 20
```

Step into the function call:

```text
(lldb) step
```

Now you are inside `add`. Inspect arguments:

```text
(lldb) frame variable a b
```

Continue:

```text
(lldb) continue
```

Quit:

```text
(lldb) quit
```

The important lesson is not the exact output. The lesson is the loop:

```text
break
run
inspect
step
inspect
continue
quit
```

---

# 4. The First Commands to Learn

Learn these first:

```text
run
breakpoint set
continue
next
step
finish
frame variable
bt
quit
```

Their common short forms are:

```text
r
b
c
n
s
finish
frame variable
bt
q
```

LLDB supports convenient aliases for many common debugger commands.

For a screen-reader workflow, the full command names are often easier to remember semantically.

---

# LLDB Command Grammar

LLDB commands usually follow this shape:

```text
noun verb --option value
```

Examples:

```text
breakpoint set --name main
breakpoint set --file src/main.rs --line 8
thread backtrace
frame variable
watchpoint set variable config
```

Many commands have short forms:

```text
b main
c
n
s
bt
```

But full names are often easier to remember and easier to hear with a screen reader.

## Help is part of the interface

Use:

```text
help
```

For a specific command:

```text
help breakpoint set
help frame variable
help watchpoint
```

LLDB’s help system is detailed. You do not need to memorize every option.

## Command completion

Start typing a command and use completion to discover subcommands:

```text
breakpoint <TAB>
```

Conceptually, LLDB exposes a command tree:

```text
breakpoint
    set
    list
    delete
    modify
    command

frame
    info
    variable
    select

thread
    list
    select
    backtrace
```

This is useful because the debugger itself can teach you its vocabulary.

## Repeat with Enter

Pressing Enter on an empty command line repeats the previous command. This is useful for paging through a backtrace:

```text
bt 5
```

Then press Enter to see more.

Be careful: an empty Enter is still a command.

---

# 5. Start the Program

Inside LLDB:

```text
run
```

or:

```text
r
```

The program starts.

If it finishes immediately, you may see something like:

```text
Process exited with status = 0
```

That means the debugger worked.

There was simply nothing that caused execution to stop.

---

# 6. Set Your First Breakpoint

A breakpoint tells LLDB:

> Stop execution here.

For example:

```text
breakpoint set --name add
```

Short form:

```text
b add
```

Then:

```text
run
```

Execution stops when `add` is entered.

LLDB officially supports setting breakpoints by function name and by file and line.

---

# 7. Break at a Rust Source Line

You can specify a file and line:

```text
breakpoint set --file src/main.rs --line 8
```

Short form:

```text
breakpoint set -f src/main.rs -l 8
```

This is useful when you already know the suspicious location.

---

# 8. List Breakpoints

Use:

```text
breakpoint list
```

This answers:

```text
What breakpoints exist?

Are they enabled?

Did LLDB resolve them?

Where are they located?
```

The distinction between a breakpoint being created and having a resolved location is important.

A breakpoint can exist before LLDB can resolve it to executable code.

---

# 9. Delete a Breakpoint

Delete one breakpoint:

```text
breakpoint delete 1
```

Delete all breakpoints:

```text
breakpoint delete
```

Be careful with the second form.

It removes the entire breakpoint set.

---

# 10. Continue

When stopped:

```text
continue
```

or:

```text
c
```

Execution resumes until:

```text
another breakpoint
a signal
a crash
program termination
```

or another debugger stop condition occurs.

---

# 11. Step Over

Suppose execution is stopped here:

```rust
let result = add(x, y);
```

Use:

```text
next
```

or:

```text
n
```

LLDB executes the current source line without entering the called function.

This is:

> Execute this line and stay at this level.

---

# 12. Step Into

Use:

```text
step
```

or:

```text
s
```

When the current line calls another function, LLDB attempts to enter that function.

For:

```rust
let result = add(x, y);
```

`step` takes you into:

```rust
fn add(...)
```

This is:

> Show me what happens inside this call.

---

# 13. Step Out

If you are inside a function and have seen enough:

```text
finish
```

This runs until the current function returns.

Conceptually:

```text
main
  ↓
add
  ↓
finish
  ↓
main
```

LLDB documents this as `thread step-out`.

---

# 14. The Most Important Difference

These three commands answer different questions:

```text
next
```

> What happens on the next line at this level?

```text
step
```

> What happens inside this function call?

```text
finish
```

> What happens after this function returns?

Learning this distinction is more important than memorizing dozens of LLDB commands.

---

# The Three Step Commands in Context

The distinction between `next`, `step`, and `finish` is the core of source-level navigation.

Suppose you are here:

```rust
let result = add(x, y);
```

## `next`

```text
next
```

Executes the line. If the line calls `add`, LLDB does not enter `add`. It stays in `main`.

Use when:

```text
you trust add
you want to see what happens after the call
you want to avoid stepping through library code
```

## `step`

```text
step
```

Executes the line. If the line calls `add`, LLDB enters `add`.

Use when:

```text
you suspect the bug is inside add
you need to inspect add’s arguments
you need to see how add transforms its input
```

## `finish`

```text
finish
```

Runs until the current function returns.

Use when:

```text
you have seen enough inside the function
you want to know what it returns
you want to return to the caller
```

The mental model:

```text
next   → stay at this level
step   → go down one level
finish → go up one level
```

---

# 15. Where Am I?

When the debugger stops, ask:

```text
Where is execution?
```

Use:

```text
thread backtrace
```

or:

```text
bt
```

A backtrace shows the call stack.

For example:

```text
frame #0  parse_config
frame #1  load_application
frame #2  main
```

Read this as:

```text
main
→ load_application
→ parse_config
```

The current location is the first frame.

LLDB's official tutorial uses `thread backtrace` to inspect the current call stack.

---

# 16. Read the Current Frame

Use:

```text
frame info
```

This gives information about the current stack frame.

You want to know:

```text
Which function?

Which source file?

Which line?

Which module?
```

This is the first orientation step after every unexpected stop.

---

# 17. Read Local Variables

Use:

```text
frame variable
```

This displays variables available in the current frame.

For a specific variable:

```text
frame variable result
```

or:

```text
frame variable config
```

This is one of the most important LLDB commands for Rust.

LLDB's `frame variable` command is designed specifically for inspecting arguments and local variables in the current frame.

---

# 18. Inspect Rust Values

Suppose:

```rust
let user = User {
    name: String::from("Alice"),
    age: 30,
};
```

At a breakpoint:

```text
frame variable user
```

You may see a structured representation of the value.

For a field:

```text
frame variable user.name
```

or:

```text
frame variable user.age
```

This lets you investigate one piece of a larger Rust value without dumping everything.

---

# 19. Inspect Nested Values

Suppose:

```rust
config.server.port
```

You can ask LLDB for:

```text
frame variable config.server.port
```

This is especially useful with configuration structures.

Instead of dumping:

```text
Config
```

inspect:

```text
config.server
config.server.port
config.server.host
```

The goal is controlled output.

For screen-reader use, smaller results are usually easier to process.

---

# 20. Arguments

Function arguments are part of the current frame.

For:

```rust
fn parse(input: &str, strict: bool)
```

use:

```text
frame variable
```

or inspect them individually:

```text
frame variable input
frame variable strict
```

Ask:

```text
What entered this function?

Is it already wrong?

Or does it become wrong here?
```

This is a fundamental debugging question.

---

# 21. Locating the First Bad Value

Suppose the final error is:

```text
invalid configuration
```

Do not immediately investigate the final error.

Trace the value backward.

Ask:

```text
Where was the value created?

Where was it transformed?

Where did it become invalid?

Which function first received the invalid value?
```

Set breakpoints at those transitions.

Then inspect the value.

The debugger becomes a tool for locating the **first incorrect state**.

---

# First Wrong State: Worked Example

Suppose this program prints the wrong result:

```rust
#[derive(Debug)]
struct Config {
    path: String,
    debug: bool,
}

fn find_config() -> Option<Config> {
    let path = std::env::var("CONFIG_PATH").ok()?;

    if path.is_empty() {
        return None;
    }

    Some(Config {
        path,
        debug: false,
    })
}

fn load_config() -> Option<Config> {
    let config = find_config()?;
    Some(config)
}

fn main() {
    let config = load_config();

    match config {
        Some(c) => println!("{}", c.path),
        None => println!("no config"),
    }
}
```

The final symptom is:

```text
no config
```

Do not start at `main`. Start at the boundary:

```text
breakpoint set --name find_config
run
```

At entry:

```text
frame variable
```

You may see nothing useful yet. Step over the `std::env::var` call:

```text
next
```

Inspect `path`:

```text
frame variable path
```

If `path` is correct, continue:

```text
next
```

Check the `if` branch. Inspect:

```text
frame variable path
```

If `path` is empty here, you have found the first wrong state.

The investigation is:

```text
main
→ load_config
→ find_config
→ env var
→ empty path
```

Now you can ask the real question:

> Why is `CONFIG_PATH` empty or missing?

That is much better than guessing at the final `None`.

---

# 22. Rust `Option`

Rust programs frequently use:

```rust
Option<T>
```

When debugging an `Option`, determine whether the value is:

```text
Some(...)
```

or:

```text
None
```

For example:

```rust
let config = find_config();
```

Stop after the call.

Then:

```text
frame variable config
```

Now you have evidence.

If it is `None`, move backward.

Ask:

```text
Why did find_config return None?
```

Set a breakpoint inside `find_config`.

Then repeat.

---

# 23. Rust `Result`

The same approach applies to:

```rust
Result<T, E>
```

Inspect the result before it is transformed.

For example:

```rust
let config = load_config()?;
```

Stop before or inside the relevant operation.

Determine:

```text
Was the result Ok?

Was it Err?

What is the error value?

Where was the error created?
```

A useful debugging chain is:

```text
Result
→ Err
→ error constructor
→ underlying cause
→ original input
```

---

# 24. `unwrap()` and `expect()`

A panic from:

```rust
value.unwrap()
```

is a particularly useful debugging location.

Set a breakpoint at the source line or run until the panic occurs.

Then inspect:

```text
value
```

The important question is not:

> Why did unwrap panic?

The important question is:

> Why was this value `None` or `Err` here?

`unwrap()` is usually the final symptom.

The cause is earlier.

---

# 25. Debugging a Panic

When Rust panics, LLDB may stop because of the underlying signal or exception behavior rather than directly at the Rust `panic!` source line.

A practical approach is:

1. Run under LLDB.
2. Observe the stop.
3. Run:

```text
bt
```

4. Inspect the current frame.
5. Move through relevant frames.
6. Identify the first frame belonging to your application.
7. Inspect its locals.
8. Trace backward toward the invalid state.

Do not assume the top frame is the root cause.

The top frame often shows where the program finally stopped.

The useful frame may be several levels below it.

---

# 26. Break on a Function

For a Rust function:

```rust
fn parse_config(...)
```

use:

```text
breakpoint set --name parse_config
```

This is usually the easiest way to investigate function entry.

When the breakpoint fires:

```text
frame variable
```

Then:

```text
bt
```

You now know:

```text
who called it
what entered it
where it was called from
```

---

# 27. Break on a Method

Rust methods may have more complicated symbol names after compilation.

Start with:

```text
breakpoint set --name method_name
```

If that is ambiguous or unresolved, investigate the symbols.

Use:

```text
image lookup --symbol method_name
```

or:

```text
image lookup --name method_name
```

The exact symbol representation can depend on compilation and mangling.

The practical rule is:

> Start with the human Rust name. If LLDB cannot resolve it, inspect the executable's symbols.

---

# 28. Break on a Source Location First

When function-name matching becomes complicated, use:

```text
breakpoint set --file src/parser.rs --line 42
```

This avoids symbol-name ambiguity.

For source debugging, file-and-line breakpoints are often the most predictable starting point.

---

# 29. Conditional Breakpoints

Suppose:

```rust
for item in items {
    process(item);
}
```

You only care when:

```text
item.id == 42
```

A conditional breakpoint can reduce noise.

The general LLDB form is:

```text
breakpoint modify 1 --condition 'condition'
```

For example:

```text
breakpoint modify 1 --condition 'item.id == 42'
```

Whether a particular Rust expression can be evaluated depends on the debug information, expression parser, optimized state, and the current frame.

When expression evaluation fails, simplify the condition or inspect the value first.

---

# 30. Do Not Confuse Source Syntax with Debugger Expression Syntax

This matters especially in Rust.

LLDB is not a Rust compiler.

The expression evaluator is not guaranteed to accept arbitrary Rust syntax.

For example, a Rust expression involving:

```text
iterators
closures
generic types
trait methods
async machinery
macros
```

may not evaluate the same way it would in Rust source.

Use debugger expressions for simple inspection.

For complicated logic:

```text
inspect the variable
step through the source
or temporarily add explicit debugging code
```

---

# 31. The `expression` Command

LLDB provides:

```text
expression
```

or:

```text
expr
```

for evaluating expressions.

For example:

```text
expr variable
```

or:

```text
expr 1 + 2
```

This can be useful, but Rust users should treat it as an advanced feature.

The debugger's expression evaluator is not a replacement for compiling Rust code.

A good progression is:

```text
frame variable
→ simple expr
→ source modification
```

---

# 32. Inspect Without Calling Code

Prefer:

```text
frame variable
```

when you simply need to inspect an existing value.

Use:

```text
expr
```

when you genuinely need expression evaluation.

This matters because evaluating an expression can execute code.

When debugging a problematic program, observation should be separated from intervention whenever practical.

---

# 33. Watchpoints

A breakpoint stops when execution reaches a location.

A watchpoint stops when memory changes.

Conceptually:

```text
Breakpoint:
"Stop here."

Watchpoint:
"Stop when this value changes."
```

LLDB supports watchpoints through the `watchpoint` command family.

A typical form is:

```text
watchpoint set variable variable_name
```

or:

```text
watchpoint set expression expression
```

Watchpoints are especially useful when you know:

> This value is correct here, but later it becomes wrong.

Then ask LLDB:

> Stop when it changes.

---

# Watchpoints in Practice

A watchpoint stops when a value changes.

Suppose:

```rust
let mut counter = 0;

for _ in 0..10 {
    counter += 1;
}
```

You can set a watchpoint after `counter` is created:

```text
breakpoint set --file src/main.rs --line 3
run
watchpoint set variable counter
continue
```

LLDB will stop when `counter` changes.

Then:

```text
bt
frame variable counter
```

This tells you:

```text
which code changed the value
what the value was before
what the value became
```

Watchpoints are most useful when:

```text
the value has a stable address
the value is not optimized away
the value is not moved frequently
```

They are less useful for:

```text
temporaries
moved values
optimized variables
values inside complex async state
```

---

# 34. Watchpoints and Rust Ownership

Rust makes mutation explicit in source, but a debugger still sees machine-level memory and debug information.

A useful investigation is:

```text
value correct
→ continue
→ value changes
→ debugger stops
→ inspect backtrace
→ identify the code responsible
```

This is particularly useful for:

```text
unexpected mutation
state changes
caches
buffers
counters
shared state
```

For values that move, are optimized away, or live inside complex structures, a watchpoint may be impractical.

Use it when the value has a stable address and LLDB can observe it.

---

# 35. Rust References and Borrows

Rust code frequently contains:

```rust
&T
&mut T
```

When debugging, distinguish:

```text
the reference
```

from:

```text
the value referenced
```

If:

```rust
fn update(config: &mut Config)
```

is stopped inside the function, inspect:

```text
frame variable config
```

Then inspect relevant fields:

```text
frame variable config.field
```

The question is:

> What state does this mutable reference currently expose?

Then step forward and determine where the mutation occurs.

---

# 36. Ownership Bugs

A compiler error such as:

```text
borrow of moved value
```

is usually better investigated first with:

```bash
rustc --explain E0382
```

rather than LLDB.

LLDB is for runtime behavior.

The compiler is for static ownership rules.

Use the tools together:

```text
Compiler
→ explains why the program cannot compile.

LLDB
→ explains what a compiled program actually does.
```

This distinction saves time.

---

# 37. Debug Assertions

Rust debug builds can contain debug assertions.

The Rust compiler enables `cfg(debug_assertions)` automatically at optimization level 0 unless explicitly configured otherwise.

This means the debug build may behave differently from a release build.

When debugging a problem, record which profile you are running.

Typical development build:

```bash
cargo build
```

Typical release build:

```bash
cargo build --release
```

If the bug only appears in release mode, reproduce it under:

```bash
rust-lldb target/release/your_program
```

Optimization can make debugging substantially harder.

---

# 38. Debugging Optimized Rust

When variables appear unavailable or source stepping behaves strangely, optimization may be involved.

A useful debugging build is:

```bash
cargo build
```

rather than:

```bash
cargo build --release
```

If you specifically need a release-like build with better debugging information, configure the Cargo profile accordingly.

For example:

```toml
[profile.release]
debug = true
```

This keeps release optimization while generating debug information.

Rust's compiler documentation distinguishes several levels of debug information and notes that removing debug information can make debugger use ineffective.

---

# 39. Backtrace as a Debugging Map

When stopped unexpectedly:

```text
bt
```

is often the first useful command.

Read it as a path:

```text
frame #0
current failure

frame #1
caller

frame #2
caller of caller

...
```

Do not immediately read every frame.

Ask:

> Which frame belongs to my code?

Then inspect that frame.

---

# 40. Selecting a Frame

Suppose:

```text
frame #0
frame #1
frame #2
frame #3
```

Select frame 2:

```text
frame select 2
```

Then:

```text
frame variable
```

Now the variables belong to frame 2.

This is extremely important.

If you inspect a variable and LLDB says it is unavailable, first ask:

> Am I in the frame where this variable exists?

---

# 41. Move Up and Down the Call Stack

You can move through frames.

Up toward the caller:

```text
up
```

Down toward the callee:

```text
down
```

Or explicitly:

```text
frame select 3
```

The conceptual direction is:

```text
callee
↑
caller
↑
caller
```

When investigating a failure, move outward until you find the frame where the relevant state was created.

---

# 42. Threads

Rust programs can use multiple threads through:

```text
std::thread
Tokio
async runtimes
thread pools
libraries
```

LLDB can inspect them.

Use:

```text
thread list
```

Then:

```text
thread backtrace all
```

The latter displays backtraces for all threads. LLDB documents `thread list`, thread selection, and `thread backtrace all` as core thread-inspection operations.

---

# 43. Select a Thread

List threads:

```text
thread list
```

Then:

```text
thread select 2
```

Now subsequent frame commands refer to that thread.

Then:

```text
bt
```

This gives you that thread's call stack.

The debugging sequence becomes:

```text
thread list
→ select suspicious thread
→ bt
→ frame select
→ frame variable
```

---

# 44. Rust and Async Debugging

Async Rust requires a different mental model.

A function such as:

```rust
async fn load_config()
```

does not execute simply as an ordinary function call from beginning to end.

It creates a future.

The executor polls that future.

Execution may suspend at:

```rust
.await
```

and resume later.

Therefore:

> A source-level async call chain is not identical to a simple synchronous stack trace.

LLDB may show executor/runtime frames between your application frames.

Do not mistake runtime machinery for the application architecture.

Look for the frames belonging to your crate.

---

# Async Debugging: Expanded Model

Async Rust is not just multithreading. It is a different execution model.

An `async fn` returns a future. The future is a state machine. Each `.await` is a possible suspension point.

When you debug async code, LLDB may show:

```text
your application frame
runtime frame
poll frame
waker frame
your application frame again
```

That is expected.

Do not try to single-step through the entire runtime. Instead, set breakpoints at application boundaries:

```text
request created
request parsed
operation started
await reached
response received
state updated
error converted
```

Then use `continue` to move between those boundaries.

A practical async debugging sequence:

```text
breakpoint set --name load_config
run
frame variable
next
step
finish
bt
continue
```

Look for frames belonging to your crate. Treat Tokio, async-std, or other runtime frames as machinery unless the bug is specifically in the runtime.

---

# 45. Debugging an `.await`

Suppose:

```rust
let response = client.get(url).await?;
```

Break before the line.

Inspect:

```text
url
client
```

Step.

If execution enters runtime machinery, use:

```text
finish
```

or continue toward the next breakpoint in your own code.

A practical strategy is to set breakpoints at meaningful application-level boundaries:

```text
request creation
request dispatch
response handling
error conversion
state update
```

rather than attempting to single-step through the entire async runtime.

---

# 46. Tokio Debugging

With Tokio, your application may run inside:

```text
runtime worker threads
scheduler code
polling machinery
waker machinery
```

Do not attempt to understand all of it immediately.

First identify:

```text
Which task?

Which application function?

Which state?

Which await boundary?
```

Then investigate only the relevant path.

For asynchronous debugging, breakpoints at your own functions are generally more useful than stepping continuously through runtime internals.

---

# 47. Debugging a Panic in Async Code

When a panic occurs:

```text
bt
```

Then separate:

```text
your application frames
```

from:

```text
Tokio/runtime frames
```

The important question is:

> Which application frame produced the invalid state?

Then inspect:

```text
frame variable
```

and trace backward.

---

# 48. Threads Versus Tasks

Do not automatically equate:

```text
thread
```

with:

```text
async task
```

A Tokio runtime may execute many tasks on a smaller number of worker threads.

Therefore:

```text
LLDB thread
≠
Tokio task
```

A thread backtrace can tell you which OS thread is executing.

It does not automatically tell you the complete logical async task history.

For async debugging, combine:

```text
LLDB
+
application logging
+
task identifiers where appropriate
+
targeted breakpoints
```

---

# 49. Debugging Tests

Rust tests are ordinary executable code from the debugger's perspective.

Build tests without running them:

```bash
cargo test --no-run
```

Then identify the test executable.

You can use Cargo's test output and artifacts to locate the executable.

Launch it with:

```bash
rust-lldb path/to/test-executable
```

Alternatively, use Cargo's debugger support when available in your environment.

The important idea is:

> A test is a small executable specification that can be debugged directly.

---

# Debugging Tests: Expanded Workflow

Build tests without running them:

```bash
cargo test --no-run
```

Cargo will print the test executables. If you need machine-readable output:

```bash
cargo test --no-run --message-format=json
```

Look for `executable` paths.

Then launch one:

```bash
rust-lldb path/to/test-executable
```

Inside LLDB:

```text
breakpoint set --name test_name
run
frame variable
bt
```

A test is just an executable. Debugging one test is usually easier than debugging the whole workspace.

If you know the test name:

```bash
cargo test test_name
```

If it fails, debug the executable that contains that test.

---

# 50. Debug One Test

Start with:

```bash
cargo test test_name
```

If it fails, determine:

```text
which package?
which test target?
which test executable?
```

Then debug that executable.

This is usually easier than debugging the entire workspace.

---

# 51. Debugging Integration Tests

Integration tests live under:

```text
tests/
```

Each integration-test file can produce its own test executable.

The workflow is:

```text
cargo test --no-run
→ identify executable
→ rust-lldb executable
→ breakpoint
→ run
```

The exact executable path can vary with the project and target configuration.

Use Cargo's build output rather than guessing the path.

---

# 52. Debugging Examples

Cargo examples can be built with:

```bash
cargo build --example example_name
```

Then:

```bash
rust-lldb target/debug/examples/example_name
```

This is useful when the example reproduces an API or integration problem more simply than the main application.

---

# 53. Debugging Binaries in a Workspace

For a workspace:

```bash
cargo build -p package_name
```

Then identify the binary produced by that package.

Run:

```bash
rust-lldb target/debug/binary_name
```

The key is to debug the smallest executable that reproduces the problem.

---

# 54. Breakpoint Strategy

Avoid setting twenty breakpoints immediately.

Start with one.

For example:

```text
breakpoint set --name parse_config
```

Run.

Inspect.

Then ask:

> What do I know now?

If necessary, add another breakpoint.

This creates a controlled investigation.

---

# 55. Breakpoint at Entry

A useful pattern is:

```text
break at function entry
→ inspect arguments
→ step
→ inspect state
→ continue
```

This answers:

> What does this function receive, and what does it do with it?

---

# 56. Breakpoint at State Change

A more advanced pattern is:

```text
break before mutation
→ inspect old state
→ step
→ inspect new state
```

This answers:

> Exactly where did the state change?

This is often more useful than breaking at the eventual failure.

---

# 57. Breakpoint at the Boundary

Many bugs become easier when debugging at boundaries.

Examples:

```text
input
→ parser

parser
→ domain object

domain object
→ database

database
→ response

response
→ user interface
```

Set breakpoints at the boundaries.

Inspect what enters and what leaves.

This is a practical form of causal debugging.

---

# 58. The First Wrong State

A powerful rule:

> **Find the first point where reality differs from your expectation.**

Suppose the final output is wrong.

Do not start at the output.

Trace:

```text
output
← transformation
← intermediate state
← input
```

At each boundary ask:

```text
Is this value correct?
```

Eventually:

```text
correct
correct
correct
wrong
```

The transition is the important location.

That is where you investigate.

---

# 59. Debugging by Hypothesis

Do not randomly step through code.

State a hypothesis.

Example:

> I think `Config::load()` returns `None` because the path is empty.

Then test it.

Set a breakpoint.

Inspect:

```text
path
```

If the path is correct:

> Hypothesis rejected.

Create the next hypothesis.

This produces:

```text
Hypothesis
→ breakpoint
→ observation
→ confirmed/rejected
→ next hypothesis
```

That is debugging.

---

# 60. Debugging a Wrong Branch

Suppose:

```rust
if config.debug {
    enable_debug();
} else {
    enable_normal();
}
```

You want to know:

> Which branch executes?

Break before the `if`.

Inspect:

```text
config.debug
```

Then:

```text
next
```

Now observe where execution goes.

You have directly answered the question.

No guessing.

---

# 61. Debugging Loops

Suppose:

```rust
for item in items {
    process(item);
}
```

Set a breakpoint inside the loop.

At each stop:

```text
frame variable item
```

Then:

```text
next
```

You can observe:

```text
iteration 1
iteration 2
iteration 3
...
```

If the failure occurs only for one item, use a conditional breakpoint or inspect the iteration state.

---

# 62. Debugging State Machines

Rust enums are often used to represent state:

```rust
enum State {
    Idle,
    Loading,
    Ready,
    Failed,
}
```

When stopped:

```text
frame variable state
```

Ask:

```text
Which variant?

How did execution reach this variant?

Which transition happens next?
```

Then place breakpoints at transition points.

This is especially useful for parsers, protocols, UI state, and asynchronous systems.

---

# 63. Debugging Enums

Enums are particularly valuable debugging landmarks.

If the program unexpectedly enters:

```text
Failed
```

trace the transition:

```text
Ready
→ operation
→ Failed
```

Then inspect the data carried by the variant.

The important question becomes:

> What event caused the state transition?

---

# 64. Debugging Traits

A trait call such as:

```rust
handler.handle(request)
```

can hide the concrete implementation.

Text search can show possible implementations.

LLDB shows what actually executes at runtime.

Set a breakpoint on candidate implementations.

Then run.

The implementation whose breakpoint fires is evidence of the runtime path.

This is particularly useful with:

```text
trait objects
dynamic dispatch
generic abstractions
plugin systems
dependency injection
```

---

# 65. Debugging Trait Objects

For:

```rust
Box<dyn Handler>
```

the concrete type is not necessarily obvious from the source line.

Inspect the value.

Then inspect the call path.

If necessary:

```text
breakpoint set
```

on candidate implementations.

Run the program.

The breakpoint that resolves and fires gives you a concrete runtime observation.

---

# 66. Debugging Generics

Generic Rust code can produce specialized machine code.

For:

```rust
fn process<T>(value: T)
```

the debugger may display instantiated types rather than the source-level abstraction you are thinking about.

When debugging generic code, ask:

```text
What concrete type is being used here?

Which instantiation am I actually executing?

What value entered this specialization?
```

The source tells you the generic design.

The debugger shows you a concrete execution.

---

# 67. Debugging Closures

Closures can produce compiler-generated names and frames.

Do not depend on memorizing their internal names.

Instead:

```text
break at the source line
→ step
→ inspect values
→ use bt
```

Treat compiler-generated frames as implementation details unless you specifically need to investigate them.

---

# 68. Debugging Iterators

Iterator chains can be difficult to single-step because Rust may represent them as nested types and optimized code.

For example:

```rust
items
    .iter()
    .filter(...)
    .map(...)
    .collect()
```

A debugger may not provide a pleasant source-level experience for every intermediate stage.

A useful debugging technique is temporarily introducing named intermediate values:

```rust
let filtered = items.iter().filter(...);
let mapped = filtered.map(...);
let result = mapped.collect::<Vec<_>>();
```

Now you have concrete source-level landmarks.

This is not a debugging failure.

It is making the program easier to observe.

---

# 69. Debugging Macros

Macros can obscure the source location.

If a macro expands into unexpected behavior:

```text
identify the macro call
→ inspect the generated behavior
→ locate the relevant expansion if necessary
→ debug the resulting function
```

For ordinary debugging, prefer the source-level macro invocation as the initial breakpoint.

Only investigate macro expansion when the expansion itself is the problem.

---

# 70. Debugging Unsafe Rust

For:

```rust
unsafe {
    ...
}
```

be especially precise.

Set a breakpoint before the unsafe operation.

Inspect:

```text
pointers
lengths
indices
references
ownership assumptions
```

Then step through the operation.

The important questions are:

```text
What invariant is assumed?

Where was the value created?

What guarantees its validity?

What changes the memory?

Where does the invalid state first appear?
```

LLDB can observe the runtime state.

It cannot prove that an unsafe invariant is correct.

That remains a source-level reasoning problem.

---

# Unsafe Rust: Expanded Context

Unsafe Rust does not mean “no rules.” It means the compiler is not enforcing all the rules for you.

When debugging unsafe code, ask:

```text
What invariant is assumed here?
Where was this pointer created?
What guarantees its validity?
What is the length?
What is the capacity?
What is the alignment?
Who owns this memory?
Who may mutate it?
When does it become invalid?
```

LLDB can show you the runtime state:

```text
frame variable ptr
memory read ptr
memory region ptr
```

But LLDB cannot prove the invariant. That is source-level reasoning.

For unsafe code, combine:

```text
LLDB
Miri
sanitizers
code review
tests
```

Miri is especially useful for detecting undefined behavior in Rust before it becomes a mysterious runtime bug.

---

# 71. Memory Debugging

LLDB can inspect memory directly.

Useful commands include:

```text
memory read
```

and:

```text
memory region
```

Use these when ordinary Rust variable inspection is insufficient.

For normal Rust application debugging, prefer:

```text
frame variable
```

first.

Move to raw memory inspection when investigating:

```text
FFI
unsafe code
pointer corruption
buffer corruption
ABI problems
allocator problems
```

---

# 72. FFI Debugging

Rust interacting with C introduces another debugging boundary.

You may see:

```text
Rust
→ extern "C"
→ C function
→ Rust callback
```

Use breakpoints at the boundary.

Inspect:

```text
Rust arguments
C arguments
return values
pointers
lengths
ownership assumptions
```

If the problem crosses the language boundary, LLDB becomes particularly useful because it can follow the native execution rather than treating Rust as an isolated world.

---

# 73. Debugging External Crates

When your code calls:

```rust
some_crate::function(...)
```

you can investigate the dependency if its debug information and source mapping are available.

Start in your code.

Set a breakpoint at the call.

Step into the dependency if necessary.

Then inspect:

```text
what your code passed
what the dependency received
where the dependency changes state
what it returns
```

Avoid immediately diving into the entire dependency.

Follow only the execution path relevant to your bug.

---

# 74. Debugging Panics from Dependencies

If a dependency panics:

```text
bt
```

Find the first frame belonging to the dependency.

Then find the frame belonging to your own crate.

Ask:

> What did my code pass into this dependency?

That is often more important than understanding the entire dependency implementation.

---

# 75. Debugging Build-Dependent Behavior

If behavior differs between:

```bash
cargo run
```

and:

```bash
cargo run --release
```

compare:

```text
features
profile
optimization
debug assertions
environment variables
configuration
platform
```

The debugger should be used after you establish which executable you are actually running.

A debugger attached to the wrong binary produces misleading evidence.

---

# 76. Confirm the Executable

Before launching LLDB, verify the binary.

For example:

```bash
ls -l target/debug/
```

Then:

```bash
rust-lldb target/debug/your_program
```

When debugging workspaces, examples, tests, or release binaries, be especially careful about selecting the correct artifact.

---

# 77. Use Cargo First

A clean workflow is:

```text
cargo check
→ cargo test
→ reproduce
→ rust-lldb
```

If the program does not compile:

```text
LLDB is not the first tool.
```

If the program compiles but behaves incorrectly:

```text
LLDB becomes useful.
```

If a test fails:

```text
debug the smallest executable that reproduces the failure.
```

---

# 78. A Basic Debugging Session

The complete beginner workflow:

```bash
cargo build
rust-lldb target/debug/my_program
```

Inside LLDB:

```text
breakpoint set --name main
run
frame variable
next
next
step
frame variable
bt
continue
quit
```

Understand each action before adding more commands.

---

# 79. A Function Investigation

Suppose:

```rust
fn parse_config(input: &str) -> Result<Config, Error>
```

Workflow:

```text
breakpoint set --name parse_config
run
```

At entry:

```text
frame variable input
```

Then:

```text
bt
```

Then:

```text
next
```

Inspect intermediate state.

If an error appears:

```text
frame variable
```

Then continue or step into the relevant function.

The investigation is:

```text
caller
→ parse_config
→ input
→ transformation
→ result
```

---

# 80. A Panic Investigation

When a program panics:

```text
run
```

Let LLDB stop.

Then:

```text
bt
```

Identify your application frame.

Then:

```text
frame select N
```

Then:

```text
frame variable
```

Ask:

```text
What state existed immediately before the failure?

Which value was invalid?

Where was that value produced?
```

Set the next breakpoint earlier in the chain.

Repeat.

---

# 81. A State-Corruption Investigation

Suppose:

```text
state is correct
```

at function A.

Later:

```text
state is wrong
```

at function D.

Do not step through every instruction.

Set breakpoints at the transitions:

```text
A
B
C
D
```

At each point:

```text
frame variable state
```

Find the first wrong state.

Then focus there.

---

# 82. A Multithreaded Investigation

Start:

```text
thread list
```

Then:

```text
thread backtrace all
```

Identify:

```text
which thread stopped
which threads are running application code
which threads belong to libraries/runtime
```

Select the relevant thread:

```text
thread select N
```

Then:

```text
bt
```

Then:

```text
frame variable
```

Do not assume the first thread listed is the important one.

---

# 83. An Async Investigation

For async Rust:

```text
break at application function
→ run
→ inspect
→ step only around important boundaries
→ continue through runtime machinery
→ stop at next application breakpoint
```

Prefer meaningful boundaries such as:

```text
request received
request parsed
operation started
awaited operation returned
state changed
error returned
```

This gives you a useful application-level execution trace.

---

# 84. LLDB Output Discipline

For screen-reader use, avoid unnecessary output.

Instead of repeatedly dumping everything:

```text
frame variable
```

ask for one value:

```text
frame variable config
```

Then:

```text
frame variable config.path
```

Then:

```text
frame variable config.debug
```

The principle is:

> **Inspect the smallest useful piece of state.**

This reduces auditory overload and makes changes easier to detect.

---

# Screen-Reader Workflow

Debugger output can be overwhelming. Use output discipline.

Instead of:

```text
frame variable
```

which may dump many variables, ask for one:

```text
frame variable config
```

Then one field:

```text
frame variable config.path
```

Then another:

```text
frame variable config.debug
```

The principle:

> Inspect the smallest useful piece of state.

Other screen-reader-friendly practices:

```text
use full command names
use help to discover syntax
use breakpoint commands to automate repetitive inspection
avoid dumping entire backtraces unless needed
use bt 5 to limit output
use frame select to move deliberately
use thread select to avoid ambiguity
```

LLDB repeats the previous command on Enter. Use this deliberately, not accidentally.

If you need to preserve output, consider redirecting LLDB output to a file or using a terminal with accessible logging.

---

# 85. Repeat Commands

LLDB repeats the previous command when you press Enter on an empty command line.

This is useful for commands that naturally produce progressive output, including backtrace inspection. LLDB documents this repeat behavior explicitly.

For example:

```text
bt 5
```

followed by Enter can continue paging through a backtrace.

Use this carefully.

An empty Enter is a command repetition, not merely a blank interaction.

---

# 86. Help Is Part of the Debugger

When uncertain:

```text
help
```

Specific command:

```text
help breakpoint set
```

or:

```text
help frame variable
```

or:

```text
help watchpoint
```

LLDB's command system is designed to expose detailed command syntax through `help`.

You do not need to memorize LLDB.

You need to know how to interrogate LLDB.

---

# 87. Command Completion

LLDB supports command completion.

Start typing:

```text
breakpoint
```

and use completion to discover available subcommands.

This is useful for screen-reader users because it lets the debugger itself expose the command hierarchy.

Think:

```text
breakpoint
    set
    list
    delete
    modify
```

rather than memorizing every command independently.

---

# 88. Breakpoint Commands

LLDB can associate commands with breakpoints.

For example, a breakpoint can automatically execute:

```text
bt
```

when hit.

The general mechanism is:

```text
breakpoint command add 1
```

Then enter:

```text
bt
DONE
```

LLDB documents breakpoint command lists as a way to automatically perform actions when a breakpoint is hit.

This can be useful for repetitive investigations.

---

# 89. Automated Observation

Suppose a function is called many times.

Instead of manually stopping every time:

```text
breakpoint
→ inspect
→ continue
```

you can attach commands to the breakpoint.

For example:

```text
breakpoint command add 1
```

then:

```text
frame variable state
bt
DONE
```

Now every hit produces the same observation.

This turns LLDB into a lightweight tracing tool.

---

# 90. When to Stop Using the Debugger

LLDB is not always the best tool.

Use ordinary logging when:

```text
you need long-running observations
you need to see many events
timing changes the behavior
the bug occurs only after a long execution
```

Use tests when:

```text
the behavior can be isolated
you need repeatability
you want a regression test
```

Use compiler diagnostics when:

```text
the problem is ownership
types
lifetimes
trait bounds
borrow checking
```

Use LLDB when:

```text
the program compiles
the problem occurs at runtime
you need to observe actual execution state
```

---

# 91. Debugging Versus Logging

Logging answers:

> What happened over time?

LLDB answers:

> What is happening here, right now?

Tests answer:

> Can I reproduce this behavior reliably?

Compiler diagnostics answer:

> Why does this Rust program violate the language rules?

Use them together.

---

# 92. Debugging as Experimental Science

A strong debugging session resembles an experiment.

Start with:

```text
Observation:
The parser returns None.
```

Hypothesis:

```text
The input path is empty.
```

Experiment:

```text
breakpoint set --name parse_config
run
frame variable input
```

Result:

```text
input is correct.
```

Hypothesis rejected.

New hypothesis:

```text
The configuration lookup fails.
```

Next experiment:

```text
breakpoint set --name find_config
continue
```

This is much more effective than randomly stepping through the program.

---

# 93. The Debugging Notebook

For difficult bugs, record four things:

```text
Observation:
What actually happened?

Hypothesis:
What do I think causes it?

Evidence:
What did LLDB show?

Next experiment:
What will I inspect next?
```

Example:

```text
Observation:
Config is None after load_config().

Hypothesis:
The config path is wrong.

Evidence:
path = "/home/user/project/config.toml"

Next experiment:
Break inside find_config().
```

This keeps the investigation externalized.

---

# 94. The Rust Debugging Loop

Use:

```text
1. Reproduce
2. Stop
3. Locate
4. Inspect
5. Trace
6. Hypothesize
7. Test
8. Fix
9. Reproduce again
10. Add a regression test
```

The final step matters.

A bug that has been fixed but cannot be reproduced by a test is easier to reintroduce.

---

# 95. From Debugging to Code Understanding

LLDB is not only a bug-fixing tool.

It can teach you how a Rust program actually executes.

Take an unfamiliar function.

Set a breakpoint.

Then observe:

```text
arguments
locals
calls
returns
branches
state changes
errors
threads
```

You are building a runtime model.

This complements static source-code investigation.

Static search asks:

> What could happen?

The debugger asks:

> What did happen during this execution?

---

# 96. Static Plus Dynamic Investigation

A strong Rust investigation combines:

```text
ripgrep
    ↓
find symbols and source locations

ast-grep
    ↓
find structural patterns

Cargo
    ↓
understand packages and dependencies

Rust compiler
    ↓
understand static constraints

LLDB
    ↓
observe runtime behavior

Git
    ↓
understand historical intent
```

No single tool provides the whole picture.

---

# 97. The Progressive Learning Path

## Level 1 — Stop and Continue

Learn:

```text
run
breakpoint set
continue
quit
```

Goal:

> Stop the program at a known location.

---

## Level 2 — Navigate Execution

Learn:

```text
next
step
finish
```

Goal:

> Control how execution moves through Rust functions.

---

## Level 3 — Inspect State

Learn:

```text
frame variable
frame info
```

Goal:

> See what the current function actually knows.

---

## Level 4 — Understand the Call Stack

Learn:

```text
bt
frame select
up
down
```

Goal:

> Understand how execution reached the current location.

---

## Level 5 — Investigate Rust Values

Practice with:

```text
struct
enum
Option
Result
references
mutable references
Vec
String
HashMap
```

Goal:

> Become comfortable inspecting real Rust state.

---

## Level 6 — Conditional Investigation

Learn:

```text
breakpoint modify --condition
watchpoint
```

Goal:

> Stop only when the interesting state occurs.

---

## Level 7 — Threads

Learn:

```text
thread list
thread select
thread backtrace all
```

Goal:

> Understand concurrent execution.

---

## Level 8 — Async Rust

Practice:

```text
async fn
.await
Tokio tasks
runtime threads
```

Goal:

> Distinguish application execution from executor machinery.

---

## Level 9 — Advanced Runtime Investigation

Learn:

```text
expression
memory read
image lookup
breakpoint commands
```

Goal:

> Investigate cases where ordinary source-level debugging is insufficient.

---

## Level 10 — Debugging as Problem Solving

Stop thinking:

> Which LLDB command should I use?

Start thinking:

> What fact do I need to establish?

Then choose the command.

---

# 98. The Core Rust Questions

When debugging Rust, repeatedly ask:

```text
Where am I?

Who called me?

What arguments did I receive?

What are the local values?

Which branch did execution take?

Which function was actually called?

What changed?

When did it change?

Who changed it?

What is the current enum variant?

Is this Option Some or None?

Is this Result Ok or Err?

What error is being propagated?

Which thread am I on?

Which application frame matters?

Where did the first incorrect value appear?
```

These questions are more important than the command vocabulary.

---

# 99. The Minimal LLDB Command Set

If you want the smallest useful working set, learn these:

```text
run
breakpoint set --name NAME
breakpoint set --file FILE --line LINE
breakpoint list
continue
next
step
finish
bt
frame info
frame variable
frame select N
thread list
thread select N
help COMMAND
quit
```

That is enough to perform a large amount of practical Rust debugging.

---

# 100. The Practical Debugging Recipe

When a Rust program behaves incorrectly:

```text
1. Reproduce the problem.

2. Build a debuggable binary.

3. Launch it with rust-lldb.

4. Set one breakpoint near the suspected boundary.

5. Run.

6. Inspect the arguments.

7. Inspect the current locals.

8. Read the backtrace.

9. Step only when necessary.

10. Find the first incorrect state.

11. Move the breakpoint closer to its origin.

12. Repeat until the cause is identified.

13. Change the code.

14. Reproduce the problem again.

15. Run the relevant tests.

16. Add or strengthen a regression test.

17. Commit only after the fix is verified.
```

---

# 101. The Central Principle

The debugger is not there to tell you what the program means.

It gives you evidence.

You supply the reasoning.

A good debugging session therefore looks like:

```text
Program behavior
→ observation
→ evidence
→ hypothesis
→ experiment
→ conclusion
```

LLDB provides the observation.

Rust provides the language and type model.

Cargo provides the build and execution environment.

The programmer connects the evidence.

---

# 102. Final Mental Model

When something goes wrong, think:

```text
Something is wrong.
        ↓
Where does the wrong behavior appear?
        ↓
What is the current state?
        ↓
Where did that state come from?
        ↓
What changed it?
        ↓
What assumption failed?
        ↓
What is the smallest correction?
        ↓
Can I reproduce the original failure?
        ↓
Can a test prevent its return?
```

The goal is not to become someone who knows hundreds of LLDB commands.

The goal is to become someone who can take an unexplained runtime behavior and systematically turn it into a verified explanation.

For Rust, that means learning to connect:

```text
source code
    +
types
    +
ownership
    +
control flow
    +
runtime state
    +
call stack
    +
threads/tasks
    +
compiler behavior
```

LLDB is the instrument that lets you observe the runtime half of that system.

**Question → Break → Inspect → Trace → Hypothesis → Verify.**

That is the core of Rust debugging with LLDB.

---

# Troubleshooting Appendix

## Breakpoint is pending or unresolved

Symptom:

```text
Breakpoint 1: no locations (pending)
```

Possible causes:

```text
wrong function name
name mangling
debug information missing
binary not loaded yet
breakpoint set before run
```

Try:

```text
breakpoint set --file src/main.rs --line 10
breakpoint list
image lookup --name function_name
```

## Variable is unavailable

Symptom:

```text
error: no variable named 'x' in this frame
```

Possible causes:

```text
you are in the wrong frame
the variable was optimized away
the variable is out of scope
the binary lacks debug info
```

Try:

```text
bt
frame select N
frame variable
```

## Stepping behaves strangely

Possible causes:

```text
optimization
inlining
async suspension
macro expansion
```

Try:

```bash
cargo build
```

instead of:

```bash
cargo build --release
```

## Expression evaluation fails

Symptom:

```text
error: expression evaluation failed
```

Possible causes:

```text
complex Rust syntax
generic types
trait methods
optimized code
no debug info
```

Try:

```text
frame variable
```

first. Use `expr` only for simple expressions.

## Threads look confusing

Remember:

```text
thread ≠ async task
```

A Tokio runtime may run many tasks on a few threads. Use `thread list` and `thread backtrace all`, but combine that with application logging and task identifiers.

## Wrong binary

Always confirm:

```bash
ls -l target/debug/
```

Then launch the exact executable you intend to debug.

---

# Glossary

```text
breakpoint
    A request for the debugger to stop when execution reaches a location.

watchpoint
    A request for the debugger to stop when a value changes.

frame
    One level of the call stack. The current frame is where execution is stopped.

backtrace
    The list of frames from the current location outward to the program entry.

thread
    An operating-system thread.

task
    An async logical unit of work. Many tasks may run on one thread.

future
    A value that represents work that may complete later.

poll
    The operation by which an executor advances a future.

debug information
    Metadata that maps machine code to source locations, types, and variables.

symbol
    A name attached to compiled code or data.

name mangling
    The compiler’s encoding of a source-level name into a unique symbol name.

optimization
    Compiler transformations that improve performance but can reduce debug accuracy.

pretty-printer
    Debugger support code that formats complex values in a readable way.

expression evaluator
    The part of LLDB that tries to evaluate expressions in the debugger.

image lookup
    An LLDB command for searching symbols and types in loaded binaries.
```

---

# Minimal Command Reference

```text
run
    Start the program.

breakpoint set --name NAME
    Break at a function name.

breakpoint set --file FILE --line LINE
    Break at a source location.

breakpoint list
    Show breakpoints.

breakpoint delete N
    Delete breakpoint N.

continue
    Resume execution.

next
    Step over the next source line.

step
    Step into the next function call.

finish
    Run until the current function returns.

bt
    Show the call stack.

frame info
    Show information about the current frame.

frame variable
    Show local variables in the current frame.

frame variable NAME
    Show one variable.

frame select N
    Select frame N.

thread list
    List threads.

thread select N
    Select thread N.

thread backtrace all
    Show backtraces for all threads.

watchpoint set variable NAME
    Stop when NAME changes.

expr EXPRESSION
    Evaluate an expression.

memory read ADDRESS
    Read memory.

image lookup --name NAME
    Search for a symbol.

help COMMAND
    Show help for COMMAND.

quit
    Exit LLDB.
```

---

# Exercises

1. Build a small Rust program with `cargo build`. Launch it with `rust-lldb`. Set a breakpoint on `main`. Run. Inspect the frame.

2. Set a breakpoint on a function with `breakpoint set --name`. Inspect its arguments with `frame variable`.

3. Set a source-line breakpoint. Step with `next`, `step`, and `finish`. Observe the difference.

4. Create an `Option<T>` value. Break after it is created. Inspect whether it is `Some` or `None`.

5. Create a `Result<T, E>` value. Break before it is unwrapped. Inspect the error.

6. Cause a panic. Run under LLDB. Use `bt` to find your application frame. Inspect its locals.

7. Write a loop. Set a breakpoint inside it. Inspect the loop variable on each iteration.

8. Write an enum state machine. Break on a state transition. Inspect the variant.

9. Write a small async function with Tokio. Break at an application boundary. Use `bt` to distinguish your frames from runtime frames.

10. Write a failing test. Use `cargo test --no-run` to find the test executable. Debug it with `rust-lldb`.

---

# Further Reading

```text
LLDB official documentation
    https://lldb.llvm.org/

LLDB tutorial
    https://lldb.llvm.org/use/tutorial.html

Rust compiler debug information
    https://doc.rust-lang.org/rustc/codegen-options/index.html

Cargo profiles
    https://doc.rust-lang.org/cargo/reference/profiles.html

Miri
    https://github.com/rust-lang/miri

Tokio debugging
    https://tokio.rs/
```

---

# The Central Context


> Debugging is observing what the program actually does.


LLDB observes machine execution.

Rust debug information translates machine execution back into Rust terms.

The translation is usually good, but not perfect.

Optimization, async, generics, traits, macros, and unsafe code can weaken the translation.

Your job is not to trust the debugger blindly.

Your job is to use the debugger to gather evidence, then reason about that evidence in Rust terms.

---

That is the context that turns a list of commands into a debugging method.

---

**Question → Break → Inspect → Trace → Hypothesis → Verify.**

That is the core of Rust debugging with LLDB.


