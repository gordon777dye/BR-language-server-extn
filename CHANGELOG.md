# Changelog

Each release is on the repository's
[Releases page](https://github.com/gordon777dye/BR-language-server-extn/releases), with its `.vsix`.

## 0.1.4 — 2026-11-25

- **Command options are checked as BR checks them,** in `EXECUTE` strings. Each command takes only
  the options its handler in BR reads, and one it doesn't is reported with BR's error: `COPY a b -A`
  and `FREE x -A` (1006, BR 5.03), `RENAME a b -R`, `MOVE a b -D`, `DIR -X`, `SEARCH x a.txt -L -C`,
  `INDEX … RE` (7605: a keyword needs three letters), `STATUS -Q` (1000), and `SYSTEM -W` and
  `SYSTEM +64` (4501, BR 5.03). A name after an option is reported too, since BR reads it as one. A
  command with no options reads a dash word as a word (`AUTO -5`).
- **Lines BR accepts are no longer reported:** `COPY`, `RENAME` and `MOVE` with one name;
  `PROTECT f BOGUS` and `SORT a b`, whose extra words BR ignores; anything after `MKDIR`, `RMDIR`
  and `VERIFY`. `INDEX` takes `SHRI`.
- **`BREAK … AFTER n`** (BR 5.03) is checked in `EXECUTE` strings: `BREAK TOTAL AFTER 5`,
  `BREAK 1200 AFTER 99 IF X>0`, `BREAK AFTER 2 IF X>100`. A system function after `BREAK`
  (`BREAK LEN`) and `DEBUG STEP OUT` check clean too.
- **FileIO layouts that share a prefix** are reported. Each one gets a warning on its prefix
  that names the others, because a subscript is a layout's prefix and a field's name, so the
  same field name in two such layouts would be one subscript for two fields. This shows in the
  editor, where an edit to one layout updates the others that are open, and in `-workspace`.
- **FileIO subscripts that name no field** are warned about (`unknown-subscript`). A numeric
  variable whose name starts with a layout's prefix but names none of its fields, such as `CO_NAMX`,
  is reported on the line you're typing. **BR: Show All Errors** reports them across the whole
  program until you close it. It checks only the program it's given. The check isn't always on,
  because a program being moved to FileIO can still have variables of its own with those prefixes.
  Setting: `br.diagnostics.fileioSubscripts`.
- **FileIO layouts are coloured** with the same kinds as BR source, so a theme colours both alike.
  Field names and key segments are variables. The prefix is a namespace and FORM codes are types.
  File names and date masks are strings, and what FileIO never reads is a comment. Descriptions
  aren't coloured. Semantic colouring is now on by default for layouts and BR source.
- **FORM codes in BR source are coloured,** as types with their sizes as numbers. This covers
  `FORM` statements and `USING "FORM …"` strings.
- **Column 4 completion follows the field's type.** A numeric field is offered `DATE(Days)`. A
  string field is offered `DATE(cymd)`, `DATE(ymd)`, `DATE(mdy)` and `DATE(mdcy)`.
- **Hovering in a layout** says what the part under the cursor is: the field, the key segment, or
  the column.
- **More layout checks**, from Appendix A of ONBOARDING.md:
  - `DATE(serial)` in column 4 is a warning, because it isn't a day number to FileIO;
  - so is a date mask on a numeric field, which should hold a BR day number, `DATE(days)`;
  - and a mask on a string field that lacks a year, a month or a day, since BR takes the missing
    part from today's date;
  - a key segment's `(start:end)` that runs past its field is a warning;
  - Information notes cover a description over 80 characters, column 4 text that FileIO ignores
    (including a subscript written there), and column 5 text that isn't a byte range.
- **A variable given a value by `READ`, `REREAD`, `INPUT`, `LINPUT` or `RINPUT`** is now
  highlighted as written there, as a `LET` target is. It was highlighted as read.

## 0.1.3 — 2026-11-24

- **Debugging (beta).** BR programs can now be debugged in VS Code and in Cursor. Give
  `DEBUG LISTEN` in BR, then start **Attach to BR**. You get:
  - line breakpoints, placed by BR's line numbers after Lexi processing;
  - conditions, hit counts and logpoints;
  - breakpoints by name: a function, a variable or a label;
  - stopping on errors;
  - stepping;
  - the call stack across `GOSUB`s, functions, libraries and `CHAIN`;
  - variables, watch and hover;
  - BR commands typed in the Debug Console as `BR <command>`.

  It needs a BR 5.03 build that has the new debugger messages, on Windows. The README's new
  **Debugging** section covers both editors, BR's limits, and the parts not yet tested.
- **TRACE (BR 5.03).** brls checks and completes BR 5.03's new forms of the statement.
  - **The modes** `LINES`, `LABELS` and `PRINT` can be given in any order, then `OFF`, or `ON`. A
    bare `TRACE`, which now means `TRACE ON`, is accepted.
  - **A trace file:** `>file` or `>>file` comes last. An unquoted name runs to the next blank and
    is the file's name as written, so `>F$` names a file and isn't taken for the variable. A
    quoted name may be a string expression.
  - **On the console,** the trace options of `RUN` and `GO` in `EXECUTE` strings are checked:
    `RUN TRACE LABELS >t.txt`, `GO TRACE PRINT OFF`.
- **FORM specifications.** In a `FORM` statement, or a `USING "FORM …"` string, the format codes
  (`C`, `PD`, `PIC(` …) are offered where a specification begins. Hovering over a code describes
  it.
- **FileIO layouts** have four more checks:
  - a key suffix other than the letters `U`, `Y`, `B` and `P` is an error;
  - a `B` or `P` suffix on a field whose FORM isn't binary or packed decimal is a warning;
  - a repeat factor in a layout row (`4*C 30`) is an error, because each field needs a row of its
    own;
  - a byte range in column 4, which is the date-format column, is a warning, since FileIO never
    reads it.
- **`WAIT=` is spelled as BR spells it** in completion and in messages. It was shown as `WAITEQ`.
  Hover now shows `WAIT=` in the syntax of `READ`, `WRITE`, `CLOSE`, `PRINT`, `INPUT`, `LINPUT`,
  `RINPUT`, and `INPUT`/`RINPUT` `FIELDS` and `SELECT`.
- **Hover links for clause keywords follow BR's exact spelling.** For example, `WAIT` and `WAIT=`
  are no longer taken for each other, and `KEY=` no longer stands in for `KEY>=`. `RESERVE` and
  `RELEASE` link to the file statements.
- **A name over 30 characters** is reported with its length. The message notes that a `$` isn't
  counted. A function name that is too long used to be reported as "every BR function name
  begins FN"; it is now reported as too long.
- **`COUNT`, `DATA`, `NOCOUNT` and `NODATA` are no longer offered as RUN and GO command qualifiers.** BR's
  table no longer has them.
- **Error descriptions:** 4178 is new (a restricted session's client didn't log in in time), and
  4156's description now says when BR raises it: from a console client, or in local mode where
  BR can't open a new session (a Mac).
- **`LIBRARY` names stay correct when workspace folders change.** brls matches a `LIBRARY` name to
  a file by the file's path within the workspace folder that holds it. That name was worked out
  when the file was first read, so adding or removing a folder could leave it out of date until
  the server restarted. Each file's name is now worked out again when the folders change. A file's
  name changes only in these cases:
  - **An open file that a newly added folder holds** is now named by its path within that folder.
    The same goes for a file in a folder added inside another one.
  - **An open file from a folder that was removed** is named by its path within a folder that
    still holds it, if one does, and otherwise by its full path.
  - **A file kept under its full path because of a clash** gets its short name back when the
    other file goes. A clash is two folders that each hold, say, `lib/x.brs`.

  No file on disk is renamed.
- **Colouring sends only what changed.** When the editor asks for it, brls sends the changes to
  the semantic colouring rather than the whole file's.
- **`br.trace.server`** now also records brls's responses and how long each request took. At
  `verbose` it records the result or the error too.

## 0.1.2 — 2026-11-22

- **The SEARCH command (BR 5.03)** is recognised in `EXECUTE` strings: its attributes, pattern
  (a word, a quoted string or a `/regex/`), file names, options and redirection are checked and
  completed. `SE`, its abbreviation, is no longer reported as an error.
- **`HELP$`** takes BR 5.03's third argument, `"BROWSE"`. With three arguments the second must be a
  string, and a number there is reported, as BR refuses it.
- **TYPE, COPY, MOVE and RENAME** with several file names are no longer reported as errors.
- **Hover links** go only to what documents the word as it is used. The `DATE` function no longer
  links to the `DATE` configuration statement, nor `POS` the function to the `POS` format code.
- **`BTREE_VERIFY`**, which BR 5.03 removed, is no longer offered as a STATUS topic.
- **Installing in Cursor** is described in the README.

## 0.1.1 — 2026-11-20

- **Your own Lexi dialect.** A project can have its own `lexi.json`, the file that spells every word
  Lexi recognises and everything it writes. **BR: Create lexi.json (Lexi dialect) in the Workspace**
  starts one from the built-in dialect; the `br.lexi.dialect` setting names another file. brls reads
  it again whenever it changes. The dialect also decides which files count as Lexi, the numbers Add
  and Strip Line Numbers use, and the words the Lexi snippets insert.
- The dialect file needs to hold only what it changes: the rest keeps the built-in values.
- A dialect file brls can't use is marked with an error on the file itself, at the key or rule
  concerned, including a misspelled key.

## 0.1.0 — 2026-11-20

The first release.

- BR language support from the brls language server: diagnostics, completion, snippets, signature
  help, hover, parameter-name hints, go to definition, references, highlights, rename, outline,
  workspace symbols, call hierarchy, document links, folding, expand selection, semantic colouring
  and quick fixes.
- Line numbers: the next number on Enter, and **BR: Add Line Numbers** / **BR: Strip Line
  Numbers**.
- Lexi source checked as the BR it becomes, and **BR: Show Lexi Expansion**. The original Lexi is
  used instead when the `crs-dev.vslang-br` extension is installed and disabled.
- Running BR from the editor: compile, run, compile on save, view compiled programs, decompile,
  search compiled programs, and BR's runtime information.
- FileIO layouts: diagnostics, hover, outline and formatting, and their subscript constants in BR
  source.
- Servers bundled for 64-bit Windows and 64-bit Linux.
