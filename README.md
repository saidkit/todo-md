# todo-md

A backlog and a todo list as one plain Markdown file each.

todo-md is a profile of GitHub Flavored Markdown (GFM) with task lists, plus a few inline extensions taken
from org-mode: a date stamp, a priority cookie, tags, ids and mentions. Every todo-md file is a valid GFM
document, so it renders as an ordinary task list on GitHub and in any editor, and a program can read it with a
small parser.

## Why

A list kept in a text file should be easy to read by hand, easy to diff, and exact enough that a tool can
change one line without disturbing the rest. A fixed format with a version number gives people and tools that
agreement, and one specification stops each tool from carrying its own copy of the rules.

## A short example

```markdown
---
format: todo-md/2
targets:
  Web: external
---
# Web
=====
## Board
- [ ] [2026-03-01] [#A] Show the stage of each entry on its card #ui @1793259681351
- [+] [2026-03-02] Filter entries by tag, ask @alice #filter @1793259681352
```

An entry is a task list item: its state (`[ ]` open, `[+]` in progress, `[X]` done), the day it was raised, an
optional priority, the text with its tags, and an id.

## Files

- `SPEC.md`: the versioned rules for both profiles, with a formal grammar, a valid and an invalid example for
  each rule.
- `examples/`: one valid file per profile.
- `CHANGELOG.md`: one entry per format version.
- `LICENSE`: Creative Commons Attribution 4.0 International (CC BY 4.0).

## Implementations

No implementation is listed yet.
