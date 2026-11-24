# Changelog

One entry per format version. A version changes only when a valid file of the previous version stops being valid
or changes meaning.

## 2

The first published version of the format.

- One profile of GitHub Flavored Markdown with task lists and a few org-mode inline extensions, for a backlog and
  for a todo list (see SPEC.md).
- Front matter carries `format: todo-md/2` on both profiles; a backlog may also declare `targets:`.
- The date stamp is `[YYYY-MM-DD]`, the priority cookie `[#A]` `[#B]` `[#C]`, the tag `#word`, the id `@` and 13
  digits, the mention `@name`.
- Under a todo line any plain note is allowed; `in <feature>`, `adr:` and `ref:` are the recognised kinds a board links.
- `[+]` is the in-progress state; it has meaning only from the profile.

Added within version 2 (a valid file stays valid and keeps its meaning):

- The author `~@name` (R13): one a line, right before the id, written by the tool that writes the line; `~@agent`
  when no person asked for the write. A missing author and `~@agent` are warnings; a malformed or second author is
  a finding.
