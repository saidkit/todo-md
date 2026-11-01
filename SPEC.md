# todo-md specification

Version: todo-md/2

todo-md is a plain-text format for a backlog and for a todo list: one Markdown file, one line per item, read by
people and by tools. It is a profile of GitHub Flavored Markdown (GFM) with task lists, plus a few inline
extensions taken from org-mode. This page is the whole specification. It has two profiles, the backlog and the
todo list, and one version number.

## 1 Conformance

- Every todo-md file is a valid GFM document (GitHub Flavored Markdown Spec, version 0.29-gfm, 2019-04-06). It
  renders as an ordinary task list on any GFM renderer.
- A file declares its version in its front matter (rule R1). A reader reads exactly the version it implements
  and reports any other version as one finding; it never guesses.
- A finding is an error unless a rule says it is a warning. A file with no error finding conforms.
- The two profiles share the front matter, the entry line and its inline extensions. They differ in their
  headings (rules R2 and R3 against R11) and in what may sit under an entry.

## 2 The GFM constructs the profile uses

Section numbers are those of the GFM Spec 0.29-gfm. A construct that is not in this table is outside the
profile: a reader keeps it as an opaque block, and where a rule below forbids it, reports one finding.

| Construct | GFM section | Use in todo-md |
|-----------|-------------|----------------|
| Front matter | not in GFM; see GFM 4.1 and GFM 4.3 | A leading `---` block. As GFM it is a thematic break followed by a setext heading, so the file stays valid; GitHub shows it as a table of its keys. The profile reads it as front matter before it reads any block. |
| Thematic break | GFM 4.1 | Delimits the front matter; a line of four or more `-` after a heading is a rule line (R2). |
| ATX heading | GFM 4.2 | The section (level 1), topic (level 2) and subtopic (level 3) headings. |
| Setext heading | GFM 4.3 | Not a profile heading. A line of `=` under a heading is a decorative rule line (R2); a setext heading anywhere else is a stray line, one finding. |
| Fenced code block | GFM 4.5 | Not allowed in a file, one finding. Fences appear only in this page's examples. |
| Paragraph | GFM 4.8 | The text of a list item; a paragraph outside a list item is a stray line, one finding. |
| List item and nesting | GFM 5.2 | An entry is a bullet list item; a sub-item is a list item nested in it. Nesting follows the list-item indentation rules: a child begins at the column where its parent's content begins, which is two columns deeper than a `- ` marker at column 0. |
| List | GFM 5.4 | A run of items at one indentation. |
| Task list item (extension) | GFM 5.3 | The `[ ]` and `[x]` marker of an entry (R5). |
| Code span | GFM 6.3 | Inline code in a line's text; its content is opaque to the extensions below. |
| Link | GFM 6.6 | A sub-item may be a link to a record or a feature (R10). |

GFM 5.3 recognises only `[ ]` (open) and `[x]` or `[X]` (checked). `[+]` is therefore not a task list marker
for any GFM renderer: it is a list item whose text starts with the characters `[+]`, and a plain renderer
shows it as such. `[+]` has meaning only from this profile (R5).

## 3 The extensions, and their origin in org-mode

Each extension is an inline construct over the text of one list item. None adds a block to GFM.

| Extension | Spelling | Origin in org-mode |
|-----------|----------|--------------------|
| Date stamp | `[YYYY-MM-DD]` | The inactive timestamp `[2026-03-01 Sun]`, written without the weekday. |
| Priority cookie | `[#A]` `[#B]` `[#C]` | The priority cookie of a headline, same spelling. |
| Progress state | `[+]` | The partial checkbox `[-]` of an org-mode list; the middle state of `[ ]`, `[-]`, `[X]`. The profile spells it `[+]` (R5). |
| Tag | `#word` | The headline tag `:word:`, written inline. |
| Id | `@` and 13 digits | The `ID` property of an entry (`:ID:`), written inline as a bare millisecond count. |
| Mention | `@name` | None. org-mode tags may start with `@` for a context (`:@home:`); the profile reads a letter-led `@name` as a person or a team. GitHub also shows it as a mention. |

## 4 The rules

Notation of the formal grammar below: EBNF in the style of ISO 14977. `,` is a sequence, `|` a choice, `[ ]` an
option, `{ }` zero or more, `( )` a group, `"..."` a terminal, `(* ... *)` a comment, `? ... ?` a description of
a set of characters, `N * x` is `x` repeated N times. The grammar describes lines; a line ends with `eol`. Every
rule of this page maps to productions here. Where the grammar cannot say it (equal indentation, uniqueness) the
rule says it in words.

```ebnf
backlog-file    = backlog-front , { blank } , { section } ;
todo-file       = todo-front , { blank | html-comment } , { todo-section } ;

(* R1, front matter and version *)
backlog-front   = "---" , eol , { fm-line } , version-line , { fm-line | targets-block } , "---" , eol ;
todo-front      = "---" , eol , { fm-line } , version-line , { fm-line } , "---" , eol ;
version-line    = "format: todo-md/" , digit , { digit } , eol ;
targets-block   = "targets:" , eol , { "  " , name , ": external" , eol } ;
fm-line         = ? any line other than "---" and other than a version-line ? , eol ;

(* R2, R3, section and topics of the backlog *)
section         = h1 , [ rule-line ] , { blank | entry | topic } ;
topic           = h2 , { blank | entry | subtopic } ;
subtopic        = h3 , { blank | entry } ;
h1              = "# " , name , eol ;
h2              = "## " , name , eol ;
h3              = "### " , name , eol ;
rule-line       = ( "=" , { "=" } | "----" , { "-" } ) , eol ;

(* R4, R5, R6, R8, the entry line *)
entry           = "- [" , state , "] " , line-text , eol , { sub-item } ;
state           = " " | "+" | "X" ;
line-text       = [ stamp , " " ] , [ priority , " " ] , text ;
stamp           = "[" , 4 * digit , "-" , 2 * digit , "-" , 2 * digit , "]" ;
priority        = "[#" , ( "A" | "B" | "C" | "High" | "Medium" | "Low" ) , "]" ;

(* R7, R12, the inline words of a text *)
text            = word , { " " , word } ;
word            = tag | id | mention | code-span | link | plain ;
tag             = "#" , letter , { letter | digit | "-" | "_" } ;
id              = "@" , 13 * digit ;
mention         = "@" , letter , { letter | digit | "-" | "_" } ;
code-span       = "`" , ? any characters except a backtick ? , "`" ;
link            = "[" , ? label ? , "](" , ? target without spaces ? , ")" ;
plain           = ? any other run of characters without a space ? ;

(* R9, R10, sub-items *)
sub-item        = indent , "- " , text , eol ;
indent          = "  " , { "  " } ;

(* R11, the todo list *)
todo-section    = h1 , [ rule-line ] , { blank | todo-entry | todo-subsection } ;
todo-subsection = h2 , [ rule-line ] , { blank | todo-entry } ;
todo-entry      = "- [" , state , "] " , line-text , eol , { todo-child } ;
todo-child      = indent , ( "[" , state , "] " , line-text | todo-note ) , eol ;
todo-note       = "- " , text ;   (* any text; "in ", "adr: " and "ref: " are the recognised kinds *)
todo-kind       = "in " | "adr: " | "ref: " ;
html-comment    = "<!--" , ? any characters up to "-->" ? , "-->" , eol ;
```

In `todo-child` a note is `indent , "- " , text` and a child box is `indent , "- [" ...`. A note whose text starts
with a `todo-kind` is a typed note that a board links; any other text is a plain note. A child sits exactly two
columns deeper than its parent.

Each rule below gives the rule, a valid example and one or more invalid examples. A reader must read each valid example with no
finding and each invalid example with exactly the finding named in its label. An example that does not start
with `---` is shown without its front matter; read it as if it were preceded by

```text
---
format: todo-md/2
targets:
  Web: external
---
```

A todo example is labelled `Todo valid:` or `Todo invalid (...)` and is read as a todo file with the same
front matter, without `targets`.

## R1 Front matter and the version key
Rule: a backlog or todo file opens with a `---` front matter that carries `format: todo-md/2`. A missing key or
any other version is one finding.  A key of another name is not a version key and is never read as one. The front matter is data for the file only: a backlog reads `format` and `targets`
(R2) and ignores every other key; a todo list reads `format`. Other keys are free text and carry no meaning.
Valid:
```text
---
format: todo-md/2
purpose: free text
---
```
Invalid (format-missing):
```text
---
purpose: no version here
---
```
Invalid (format-unknown):
```text
---
format: todo-md/3
---
```
Todo invalid (format-missing):
```text
---
purpose: no version here
---
# lane
- [ ] [2026-03-01] a todo file whose front matter has no version
```

## R2 Section
Rule: the top level of a backlog is a section: a level-1 ATX heading `# <name>`, optionally followed by a rule
line (a line of `=`). The name is declared under `targets:` in the front matter (`<name>: external`) or is a name
the reading tool knows from its own environment, such as the components of a workspace. Any other name is one
finding. A rule line is decoration: it is a paragraph of `=` signs to a plain renderer and is ignored.
Valid:
```text
---
format: todo-md/2
targets:
  Web: external
---
# Web
=====
- [ ] [2026-03-01] a target entry
```
Invalid (section-undeclared):
```text
# Nowhere
=========
- [ ] [2026-03-01] x
```

## R3 Two topic levels
Rule: under a section a level-2 heading is a topic and a level-3 heading under it a subtopic. Nothing deeper is a
heading: a level-4 heading is one finding, and deeper items are sub-items (R9). A numbered list item used as a
group (`1. Name`) is one finding.
Valid:
```text
# Web
=====
## Board
### Backlog view
- [ ] [2026-03-01] x
```
Invalid (topic-depth):
```text
# Web
=====
## A
### B
#### C
- [ ] [2026-03-01] x
```
Invalid (group-numbered):
```text
# Web
=====
1. A group
- [ ] [2026-03-01] x
```

## R4 Entry line
Rule: an entry is one bullet list item `- [<state>] <text>` at column 0, a direct child of the document, under a
section, topic or subtopic. Its first line holds the stamp, the priority and the text (R6, R8, R7). An entry at
another indent is one finding, and so is a box nested under an entry. The indent is spaces; a tab in the indent
is one finding. A file holds no fenced code block.
Valid:
```text
# Web
=====
## A
- [ ] [2026-03-01] an entry at column 0
```
Invalid (entry-indent):
```text
# Web
=====
  - [ ] [2026-03-01] an entry at the wrong indent
```
Invalid (nesting-depth):
```text
# Web
=====
- [ ] [2026-03-01] a parent
  - [ ] [2026-03-02] a box nested under it
```

## R5 State marks
Rule: the state is `[ ]` open, `[+]` in progress, `[X]` done. GFM 5.3 knows only `[ ]` and `[x]`: the profile
reads a lowercase `[x]` as done and reports one finding (write `[X]`); `[+]` is meaning the profile gives to a
list item whose text starts with `[+]`. Any other mark is one finding.
Valid:
```text
# Web
=====
- [ ] [2026-03-01] open
- [+] [2026-03-02] in progress
- [X] [2026-03-03] done
```
Invalid (state-lowercase):
```text
# Web
=====
- [x] [2026-03-01] done with a lowercase mark
```
Invalid (state-unknown):
```text
# Web
=====
- [?] [2026-03-01] an unknown mark
```

## R6 Stamp
Rule: `[YYYY-MM-DD]` is the day the item was raised, an org-mode inactive date without weekday, and it follows
the state mark. A stamp of any other form, such as `[@260301]`, is one finding. An entry without a stamp is read
and is a warning. A priority before the stamp is one finding. A stamp does not belong in a sub-item.
Valid:
```text
# Web
=====
- [ ] [2026-03-01] ask @alice about it
```
Invalid (stamp-former):
```text
# Web
=====
- [ ] [@260301] a stamp in another form
```
Invalid (stamp-missing):
```text
# Web
=====
- [ ] an entry without a stamp
```
Invalid (token-before-stamp):
```text
# Web
=====
- [ ] [#A] [2026-03-01] the priority comes after the stamp
```
Invalid (stamp-in-sub-bullet):
```text
# Web
=====
- [ ] [2026-03-01] an entry
  - [2026-03-02] a stamp in a note
```

## R7 Tags
Rule: a `#word` in the text of a backlog entry is a tag: a letter first, then letters, digits, `-` or `_`. Tags
are matched exactly and are not part of the entry's words. In a todo list `#word` stays in the text. The stage of
an entry (open record, promoted feature) is derived from its links (R10) and is never written: a bracketed word
naming a stage, such as `[Scoped]` or `[Done]`, is not part of the format and is one finding.
Valid:
```text
# Web
=====
- [ ] [2026-03-01] tidy the reader #scoped #ui
```
Invalid (stage-former):
```text
# Web
=====
- [ ] [2026-03-01] [Scoped] tidy the reader
```

## R8 Priority
Rule: `[#A]`, `[#B]` or `[#C]`, after the stamp. `[#High]`, `[#Medium]` and `[#Low]` are read as the same three
values. Any other letter or word is one finding. `[P1]` or `[P2]` is not a priority and is a warning.
Valid:
```text
# Web
=====
- [ ] [2026-03-01] [#A] first
- [ ] [2026-03-02] [#Low] second
```
Invalid (priority-letter):
```text
# Web
=====
- [ ] [2026-03-01] [#D] a letter outside A to C
```
Invalid (priority-p-scale):
```text
# Web
=====
- [ ] [2026-03-01] [P1] not a priority
```

## R9 Sub-items
Rule: a sub-item is a `- ` list item nested in an entry, two columns deeper, and nested further by two more
columns per level. An entry has at most two sub-items at its first depth.
Valid:
```text
# Web
=====
- [ ] [2026-03-01] an entry
  - a note
    - a nested note
  - a second note
```
Invalid (sub-bullets-over-two):
```text
# Web
=====
- [ ] [2026-03-01] an entry
  - one
  - two
  - three
```

## R10 Links
Rule: a sub-item may be a link `[label](target)`. A link to a record is `[<YYMMDD>-<kind>-<slug>](backlog/<same>/idea.md)`
and its label is the slug; a link to a feature is `[<FEAT-NN>](<path>)`. A relative target must exist. A label
that is not the slug of its record, or a relative target that does not exist, is one finding.
Valid:
```text
# Web
=====
- [ ] [2026-03-01] an entry with a record
  - [260101-idea-demo](backlog/260101-idea-demo/idea.md)
```
Invalid (link-label-not-slug):
```text
# Web
=====
- [ ] [2026-03-01] an entry with a wrong label
  - [demo](backlog/260101-idea-demo/idea.md)
```
Invalid (link-target-missing):
```text
# Web
=====
- [ ] [2026-03-01] an entry with a missing record
  - [260199-idea-gone](backlog/260199-idea-gone/idea.md)
```

## R11 The todo profile
Rule: a todo list has the front matter of R1 with no `targets`, and shares R5, R6 and R8 with the backlog. Its
headings are a level-1 section with an optional rule line and an optional level-2 subsection. Boxes nest by two
columns: a child box sits exactly two columns deeper than its parent. Under a todo line any plain note is
allowed; `in <feature>`, `adr: <id>` and `ref: <path or url>` are the recognised kinds that a board links. An HTML comment on its own line is ignored. A `#word` stays in the text.
Todo valid:
```text
# lane
======
- [ ] [2026-03-01] [#B] a line
  - in PROJ-01, the footer
  - waiting on the design review
  - [+] [2026-03-02] a nested line
## pickups
---------
- [X] [2026-03-03] a pickup
```
Todo invalid (priority-p-scale):
```text
# lane
======
- [ ] [2026-03-01] [P1] not a priority
```
Todo invalid (child-indent):
```text
# lane
======
- [ ] [2026-03-01] a parent
    - [ ] [2026-03-02] a sub-box four columns under it
```

## R12 Ids and mentions
Rule: an entry or a todo line carries one id, `@` and the 13 digits of a Unix time in milliseconds, written as a
whole word at the end of its text, before any sub-item (for example `@1793259681351`). The tool that writes the
line chooses the id at write time; it is never computed from the text. It counts in the text's length. An id is
unique in its file. A line with no id is read and is a warning; an id that is not 13 digits, or an id twice in
one file, is one finding. A word that starts with `@` and a digit is always an id and never a mention; a word
that starts with `@` and a letter is a mention and has no other meaning.
Valid:
```text
# Web
=====
- [ ] [2026-03-01] ask @alice about it #ui @1793259681351
```
Invalid (id-malformed):
```text
# Web
=====
- [ ] [2026-03-01] a short id @17932596
```
Invalid (id-duplicate):
```text
# Web
=====
- [ ] [2026-03-01] first @1793259681351
- [ ] [2026-03-02] second @1793259681351
```
