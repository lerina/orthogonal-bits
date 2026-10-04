# THE SWEEP
### Tactile navigation technique, applied to the command line

*A field guide for reading unfamiliar code the way a long cane reads unfamiliar ground — not by looking at the whole space at once, but by probing it, one deliberate contact at a time, and building a map from what each contact reports back.*

---

## Why Touch Beats Looking

A cane doesn't show you a room. It tells you, one tap at a time, what's actually under the tip right now — and trusts you to build the shape of the space out of a sequence of honest, local reports instead of one unreliable overview.

A terminal works the same way, if you let it. `rg`, `less`, `hx`, `ast-grep`, and `rust-lldb` don't render a codebase. Each one reports back exactly what's under its own particular kind of contact — a line of text, a window of context, a syntax tree, a running stack frame — and the skill isn't in any one tool. It's in choosing the right contact for the ground you're actually standing on.

Five techniques, five tools, each one doing for a codebase what its cane counterpart does for a sidewalk.

---

## 1. The Arc Sweep — `rg`

A cane's basic sweep moves in a wide, steady arc ahead of each step, left to right, covering the ground you're about to enter before you commit to it. It isn't trying to find anything specific yet. It's establishing that the space ahead is walkable at all.

`rg` is the arc sweep of a codebase — fast, wide, repo-scale, run before you've committed to any one file.

```bash
rg 'process_data'
```

Sweeps the whole tree in one pass. No file chosen yet, no assumption about where the answer lives — just contact with everywhere the name appears, reported back flat.

Narrow the arc without losing its width:

```bash
rg -t rust 'process_data'
rg 'process_data' -g '*.rs'
```

The sweep stays wide. It just stops reporting contact with ground you already know isn't relevant — test fixtures, vendored dependencies, generated files.

```bash
rg -l 'process_data'
```

Filenames only — the sweep telling you *which rooms* have this feature in them, before you've decided to enter any of them.

---

## 2. Constant Contact — `less` and `hx`

In constant contact, the cane tip never leaves the ground. It's the technique for ground you can't predict well — the tip rides the surface continuously, reporting every texture change as it happens, rather than sampling the ground in discrete taps.

`less` and `hx` are constant contact. Once you're inside a file, you stop sampling and start reading continuously — line after line, with nothing lost between reports.

```bash
less src/parser.rs
```

Page through with continuous control — space to advance, `b` to go back, `/pattern` to search forward inside the file you're already standing in, `n` to repeat the search without losing your place.

```bash
hx src/parser.rs:42
```

Opens directly at the line a sweep already found, dropping you into constant contact exactly where the arc sweep's report said to look.

Inside Helix, the search stays local — you're not sweeping the repo anymore, you're reading the one room you're already in:

```text
/parse_expression
n
N
```

---

## 3. Two-Point Touch — Context Flags

Two-point touch taps left, then right, alternating — checking what's immediately on either side of the path rather than only what's dead ahead. It catches the obstacle at your shoulder, not just the one at your toes.

`rg`'s context flags are two-point touch for a match: what's immediately before it, what's immediately after.

```bash
rg -B 5 'pub fn process_data'
```

Five lines of touch to the left of the match.

```bash
rg -A 15 'pub fn process_data'
```

Fifteen lines of touch to the right.

```bash
rg -C 5 'pub fn process_data'
```

Both sides at once — confirming the match is actually what you think it is before you commit to opening the file around it.

---

## 4. The Pencil Grip — Precise Search in Dense Ground

The pencil grip brings the cane in close to the body, for crowded or cluttered terrain where a wide sweep would just report noise — too many legs, too many obstacles, not enough useful signal in a wide arc. The grip narrows on purpose, trading coverage for precision.

Use the pencil grip on a codebase when a plain sweep returns too much:

```bash
rg 'fn' -t rust
```

Too wide — every function in the tree reports contact.

```bash
rg -w 'fn\s+process_data\b' -t rust
```

Narrowed. Whole-word, specific name, no stray partial matches from `process_data_legacy` or `old_process_data_handler` crowding the result.

Inside Helix, the pencil grip is a textobject — selecting exactly the function, not the ten lines around it:

```text
mif
```

Select inside the function. Precise contact, nothing extra swept up with it.

---

## 5. Touch and Slide — Following a Reference Chain

Touch and slide finds a vertical surface — a wall, a railing — and follows it, sliding the tip along the edge until it reports an opening. It's not a wide search. It's a committed trace along one specific edge until that edge actually goes somewhere.

Following an import or a reference chain is touch and slide:

```bash
rg -n '^use\s+crate::parser'
```

Found the wall — who's importing the module.

```bash
rg -n 'Parser::new'
```

Slide along it — where the wall actually opens, where the thing gets constructed.

```bash
rg -n -A 20 'fn new' src/parser.rs
```

Through the opening — reading what's actually on the other side once the slide finds the door.

---

## 6. The Curb Detection — Type Boundaries and `unsafe`

A cane finds a curb before a foot does. The tip drops, reports the change in level, and the body stops — not out of fear, just out of procedure. You don't step down blind. You check the drop, then you commit.

In Rust, the curb is any boundary where the ground genuinely changes under you — a type conversion, a `Result` that can fail, an `unsafe` block where the compiler's normal guarantees stop applying.

```bash
rg -n '\bunsafe\b' -t rust
```

Every curb in the file, found before you're standing on one by accident.

```bash
rg -n '\.unwrap\(\)|\.expect\(' -t rust
```

Every spot where a failure is assumed away instead of handled — a drop-off nobody bothered to mark.

Detecting the curb doesn't mean avoiding it. It means knowing it's there before your weight is already on it.

---

## 7. The Echo Off the Building — `ast-grep outline`

Long before the cane touches a building, its presence changes the sound around you — a change in echo, a change in air pressure, a sense of mass ahead that arrives before any direct contact does. Experienced cane users read this constantly, often without consciously noticing they're doing it.

`ast-grep outline` is that echo for a source file — the shape of the thing, reported before you've read a single line inside it.

```bash
ast-grep outline src/parser.rs
```

The mass of the file, reported back: what's in here, roughly how it's organized, before you've committed to reading any of it directly.

```bash
ast-grep outline src/parser.rs --view signatures
```

A sharper echo — not just that something's there, but the shape of its edges.

```bash
ast-grep -p '$OBJECT.unwrap()'
```

A structural sweep, same idea as `rg`'s arc but reporting contact with a *shape* instead of a literal string — every place code resembling `something.unwrap()` exists, regardless of what the something is actually called.

---

## 8. Three-Point Touch — `rust-lldb` on Unstable Ground

Three-point touch is the technique for genuinely uncertain terrain — gravel, a construction zone, a slope that might not hold weight the same way twice. Left, middle, right, middle again. More contact, more often, because the ground itself can't be trusted to behave the same way between steps.

A running program is unstable ground in exactly this sense — its state changes underfoot, which is the whole reason static reading eventually hits a limit.

```bash
rust-lldb ./target/debug/your_binary
```

Steps onto the ground itself, rather than reading a description of it.

```text
b your_crate::parser::process_data
r
```

A breakpoint — a deliberate, chosen point of contact, placed before you start moving, so the ground stops being a surprise the moment you actually reach it.

```text
n
s
frame variable
```

Next, step, inspect — the three points of contact, repeated, each one confirming the last before trusting the next.

```text
bt
```

The full trail behind you — how you actually got to the ground you're currently standing on, in case the path back out matters as much as the one forward did.

---

## Quick Reference

| Cane technique | Terrain | Tool | Use |
|---|---|---|---|
| Arc sweep | Open ground ahead | `rg` | Repo-wide search, no file chosen yet |
| Constant contact | Inside a known room | `less` / `hx` | Continuous reading once a file is open |
| Two-point touch | Either side of a step | `rg -B/-A/-C` | Context around a match |
| Pencil grip | Crowded, cluttered ground | `rg -w`, `hx` textobjects | Precise search, noise excluded |
| Touch and slide | Following a wall to a door | `rg` chained searches | Tracing an import or reference chain |
| Curb detection | A level change underfoot | `rg 'unsafe'`, `.unwrap()` searches | Finding boundaries before standing on them |
| Echo off a building | Mass sensed before contact | `ast-grep outline`, `ast-grep -p` | Structural shape, sensed before reading |
| Three-point touch | Unstable, shifting ground | `rust-lldb` | Runtime state, checked repeatedly as it changes |

No technique replaces the others. A good sweep still ends in constant contact once you've actually found the room. A clean echo off a building still deserves a curb check before you walk through whatever door it implied was there. The ground decides the technique. The technique was never the point — reading the ground accurately always was.
