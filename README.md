# Business Rules! (brls) for Visual Studio Code

Language support for **Business Rules! (BR)** in VS Code, from **brls**, the BR language server.
The extension is a launcher. It starts brls, binds its commands and keys, and contributes its
settings. Everything it knows about BR is in brls: diagnostics, completion, snippets, line numbers
and Lexi.

- **Publisher:** Application Development Systems. The extension's id is
  `application-development-systems.brls`.
- **Download:** the `.vsix` attached to the newest release on the
  [Releases page](https://github.com/gordon777dye/BR-vscode-extn-2/releases/latest). Each release
  is listed in [CHANGELOG.md](CHANGELOG.md).
- **Requires:** VS Code 1.91 or later.
- **Bundled servers:** brls for 64-bit Windows and for 64-bit Linux. On any other platform, point
  [`br.server.path`](#the-server) at a brls you have built.

---

## Contents

- [Installing](#installing)
- [Files the extension handles](#files-the-extension-handles)
- [Diagnostics](#diagnostics)
- [Writing code](#writing-code)
- [Navigating](#navigating)
- [Line numbers](#line-numbers)
- [Lexi](#lexi)
  - [The original Lexi](#the-original-lexi)
  - [Customizing Lexi: the dialect file](#customizing-lexi)
- [Running BR from the editor](#running-br-from-the-editor)
- [FileIO layouts](#fileio-layouts)
- [Commands and keys](#commands-and-keys)
- [Settings](#settings)
- [The server](#the-server)
- [Troubleshooting](#troubleshooting)
- [License](#license)

---

## Installing

1. Open the [newest release](https://github.com/gordon777dye/BR-vscode-extn-2/releases/latest)
   and download its `.vsix`, named `brls-<version>.vsix` (for example `brls-0.1.1.vsix`).
2. Install it in either of these ways:
   - **From a terminal:** `code --install-extension brls-<version>.vsix`
   - **In VS Code:** open the Extensions view, then **…** (Views and More Actions) →
     **Install from VSIX…**, and choose the file.
3. Reload the window when VS Code asks.

**If you use the `crs-dev.vslang-br` extension, disable it.** Both extensions provide the BR
language, so with both enabled every BR file gets two grammars and two language servers. brls warns
you at startup if it finds that extension enabled. Disabling it is better than uninstalling it if
you want the original Lexi (see [The original Lexi](#the-original-lexi)).

**Updating.** The extension doesn't update itself. To move to a new version, download that
release's `.vsix` and install it the same way: it replaces the version you have, and your settings
are kept. [CHANGELOG.md](CHANGELOG.md) says what each version changed. To be told when a new
version is out, use **Watch → Custom → Releases** on the repository's GitHub page.

**Removing.** Uninstall it from the Extensions view.

---

## Files the extension handles

| File | Language | Notes |
|---|---|---|
| `*.brs`, `*.wbs` | BR (`br`) | BR source, or Lexi source: see [Lexi](#lexi). |
| `*.lay` | FileIO layout (`lay`) | See [FileIO layouts](#fileio-layouts). |
| Files directly inside a folder named in `br.filelay.folders` (`filelay` by default) | FileIO layout | FileIO layouts usually have no extension. The extension opens a file there as a layout when its extension is `br.filelay.extension` (none by default). |

**Defaults set for BR files.** The extension sets two editor settings for the `br` language, and
you can override either in your own settings:

- **`files.encoding: cp437`.** BR source is CP437, so box-drawing and accented characters read and
  save correctly.
- **`editor.formatOnType: true`.** brls numbers the line Enter opens (see
  [Line numbers](#line-numbers)), and this is what lets it.

**Colouring.** A small grammar colours line numbers, strings and comments as soon as a file opens.
brls's semantic colouring then takes over. It tells apart what spelling alone can't: a variable
from a function of the same name, a label from a variable, a parameter from a module variable, and
Lexi constructs from BR.

**The workspace.** On startup brls indexes every source file in the workspace, so links between
programs (`LIBRARY` statements, functions defined in another file) work for files you haven't
opened. Four settings control the index:
- `br.sourceSuffixes`: which files are source, and how a `LIBRARY` name maps to a file.
- `br.ignoreDirectories`: folders skipped.
- `br.maxIndexedFiles`: an upper bound, so opening a very large folder by mistake doesn't scan
  forever.
- `br.closedWorkspace`: tells brls the workspace is the whole project. Only then does it report a
  `LIBRARY` program that can't be found.

---

## Diagnostics

brls checks each file as you type and reports problems in the editor and the **Problems** panel.
Each diagnostic carries a rule id, such as `unknown-function`, as its code.

The rules fall into groups, and each group is switched on or off with one setting,
`br.diagnostics.<key>`. All are on by default except `unused`.

| Key | What it reports |
|---|---|
| `syntax` | A line BR won't compile; a line over 2,000 characters; duplicate or out-of-order line numbers; more than one `OPTION`; a `!:` continuing onto a numbered line; a label with no statement; empty parentheses in a call or `DEF`; a left side of `=` that isn't a variable; a statement that compiles and does nothing |
| `unclosedBlocks` | `FOR` without `NEXT` and the reverse, `DO` without `LOOP` and the reverse, block `IF` without `END IF`, nested `IF` problems |
| `undefinedTargets` | A `GOTO`, `GOSUB` or other branch to a line or label that doesn't exist, and duplicate labels |
| `unknownFunctions` | Unknown, unimplemented and deprecated functions; duplicate functions and parameters; an unclosed `DEF`; wrong argument counts or types for an intrinsic or a user function; a literal passed by reference |
| `libraries` | A `LIBRARY` function that the program named doesn't export, or that isn't defined, and a `LIBRARY` program that can't be found |
| `formats` | A bad `USING` format, and `USING` naming something that isn't a `FORM` |
| `executeStrings` | The text of an `EXECUTE` statement or command that BR would reject, or that ends the program |
| `reservedNames` | A reserved word, or an intrinsic's name, used as a variable, array or I/O target |
| `shadowsIntrinsic` | A user name that hides an intrinsic |
| `concatTypeMismatch` | A number in a string concatenation |
| `subscriptTypeMismatch` | A string used as a subscript |
| `mixedAndOr` | `AND` and `OR` mixed without parentheses |
| `surprisingGrouping` | Exponents and negation that group differently from what they look like |
| `notAnOperator` | Something that looks like an operator but isn't one in BR, an argument BR ignores, an expression that is always zero |
| `defLibraryComma` | A stray comma in a `DEF LIBRARY` |
| `unused` (off by default) | Unused `LIBRARY` names, parameters and `DIM`s |

[SETTINGS.md](SETTINGS.md) lists every rule id under its key.

**Quick fixes.** Two diagnostics offer a fix through the light bulb, or Ctrl+.:
- `unused-library` removes the name from its `LIBRARY` statement.
- `unused-dim` removes the item from its `DIM`.

Neither is offered when the name is the only one in its statement, because the fix would leave an
empty statement.

**Errors from BR itself.** When you compile with BR (see
[Running BR](#running-br-from-the-editor)), BR's own error is shown on the line it names, as a
diagnostic from `BR`. It stays there until you next edit the file.

**Documentation links.** brls can link a diagnostic's code to the page of the BR reference that
justifies it, using `br.referenceRoot`, the folder holding that reference. Without one, the
reference is named in the message text instead.

---

## Writing code

**Completion.** What's offered depends on where the caret is, because in BR a keyword is only a
keyword in its place:

- **Where a statement can start** (an empty line, after a line number, after a `:` ending a
  statement, after `THEN` or `ELSE`), you're offered BR's statement keywords and the names in your
  file. Names are offered here because BR has an implied `LET`.
- **Everywhere else**, you're offered your file's names, BR's intrinsic functions, and the clause
  keywords the statement allows at that exact point. They come from BR's own syntax tables, so the
  list narrows as you type. For example, `OPEN #1: F$, EXTERNAL ` offers `INPUT`, `OUTPUT` and
  `OUTIN`.

Your own names sort first.

**Snippets** are offered as completions where a statement can start.
- Each snippet is checked against BR when brls is built, so none inserts code BR rejects.
- In a numbered file, a snippet of several lines numbers its lines the way Enter would, and
  indents nested lines. If there's no room for the numbers before the next numbered line, the
  snippet isn't offered.
- Snippets for the FileIO library are off by default. Turn them on with `br.snippets.fileio` in a
  project that uses FileIO.

**Signature help** appears when you type `(` or `,` in a call.
- For an intrinsic, it shows each form of the function and highlights the parameter you're on.
- For your own `FN` function, it shows its `DEF`, whether that's in this file or in the program a
  `LIBRARY` statement links. A `;` marks where the optional parameters begin.

**Hover** shows:
- a variable's shape and scope;
- a `DEF`'s signature;
- the file a `LIBRARY` link resolves to;
- an intrinsic function's description;
- a statement keyword's syntax, from BR's own syntax tables.

**Parameter names in calls (inlay hints).** At a call to your own function, each argument is
labelled with the parameter it binds to, because BR has no named arguments. A label is left out
where the argument is a variable already spelled like the parameter.

---

## Navigating

| Feature | How to use it | What it does |
|---|---|---|
| Go to definition / declaration | F12, or Ctrl+click | Jumps to where a variable, function, label or line number is defined, following a `LIBRARY` link into the defining file whether or not it's open. In BR a name is declared where it's defined, so both go to the same place. |
| Find all references | Shift+F12 | Finds every use in the file. For a function defined with `DEF LIBRARY`, it also finds the callers in other files. |
| Highlight | Put the caret on a name | Highlights every use of the name in the file, telling reads from writes. |
| Rename | F2 | Renames across every file involved (see below). |
| Outline and breadcrumbs | The Outline view | Shows functions with their parameters, linked functions, labels and declared variables. |
| Go to symbol in workspace | Ctrl+T | Finds functions and labels across the project. |
| Call hierarchy | Shift+Alt+H | Shows who calls an `FN` function, across files for a linked one, and who `GOSUB`s a line or label. A `GOTO` is a branch, not a call, and isn't listed. |
| Document links | Ctrl+click the program name in a `LIBRARY` statement | Opens the file it resolves to. There's no link where brls can't resolve it. |
| Folding | The arrows in the margin | Folds `FOR`/`NEXT`, `DO`/`LOOP`, block `IF`/`ELSE`/`END IF`, multi-line `DEF`s, and runs of comment lines. The closing line stays visible. |
| Expand selection | Shift+Alt+→ | Grows the selection outwards: a name, its expression, the statement, the line, the enclosing block, the file. |

**How rename works:**
- `A`, `A$` and `MAT A` are separate names, and a parameter stays separate from a module variable
  of the same name.
- Labels move with the branches to them.
- Renaming an exported function updates its `DEF` and every file whose `LIBRARY` statement links
  it.

brls refuses a rename, and says why, when:
- the target is a line number;
- the new name is reserved, or the wrong type (a `$` kept or dropped, `FN` missing from a function
  name);
- the new name is already in use, or a parameter would capture it;
- a file has a line that doesn't parse and mentions the name.

---

## Line numbers

**Enter numbers the next line.** In a numbered file, pressing Enter at the end of a numbered line
starts the new line with the next number.
- The step is read from the two nearest numbered lines and rounded down to 1, 2, 10, 20 or 100. It
  shrinks further, if it must, to fit before the next numbered line.
- Inside a `!:` continuation the new line is indented instead.
- Enter after the last line of a continuation numbers from the line the continuation started on.

These settings control it:
- `br.autoLineNumbers.enabled` turns it on or off.
- `br.autoLineNumbers.increment` sets the step used where the lines above give none.
- `br.autoLineNumbers.zeroPadding` sets the width where the line above gives none, from 1 to 5.

It relies on `editor.formatOnType`, which the extension turns on for BR files.

**BR: Add Line Numbers** (Ctrl+Alt+3) numbers every line of a file as Lexi would, and does nothing
else: Lexi constructs stay as you wrote them. **BR: Strip Line Numbers** (Ctrl+Alt+4) removes the
numbers. A file numbered and then stripped comes back as it was. Strip warns you when the file
branches to a line number, since those branches break once the numbers are gone. Each command is
one edit, so a single undo reverses it, and neither saves the file.

---

## Lexi

Lexi lets a BR program be written without line numbers, and with a few constructs BR doesn't have.
It's expanded into plain, numbered BR before BR loads it. brls has its own Lexi built in, so you
need nothing else installed.

**Which files are Lexi.** A Lexi file has the same `.brs` or `.wbs` suffix as BR source, so brls
decides from what's in it. A file is Lexi if it contains a Lexi construct, or has no numbered line
at all.
- If the file's first non-blank line is numbered, the file keeps its numbers and only its
  constructs are expanded.
- Otherwise every line is numbered.

**The constructs:**

| Construct | Example | Becomes |
|---|---|---|
| Append | `let A$&="tail"` | `let A$(inf:0)="tail"`, BR's append to the end of a string |
| Join with the next line | `... !_` | The two lines as one |
| Continuation | A line ending in an operator such as `+`, `,`, `&`, `(`, `and` or `or` | Joined with the lines that follow, until a line doesn't end in one. A `!:` after an operator joins the same way, since BR rejects an operator before `!:`. |
| Template string | `` `Total {{N}}` `` | `"Total "&str$(N)&""` |
| Block comment | `/* ... */` | `!` comment lines |
| Substitution | `! #DEFINE# [[Max]] = 10`, then `[[Max]]` | `10`, wherever the name is used |
| Case branches | `#SELECT# X #CASE# 1 # 2` … `#CASE ELSE#` … `#END SELECT#` | `if`/`else if`/`else`/`end if`. A `#SELECT#` can sit inside a `#CASE#`. |
| Numbering | `! #AUTONUMBER# 1000,10`, labels `L1500:` | Line numbers from 1000 in steps of 10. A label line jumps the numbering to its number when that number is still ahead; one already passed is numbered on, the step halved to catch up |

**A Lexi file is checked as the BR it becomes.**
- Every feature above works on it: diagnostics, completion, hover, navigation, rename, folding and
  colouring.
- Positions are mapped back to the line and column you typed.
- Text that Lexi writes counts as part of the construct it came from. So an edit that would change
  it, such as a rename reaching into a `#DEFINE#` value, is refused.

**Problems in Lexi source are errors.** brls reports Lexi source in error, and the expansion leaves
it as written rather than guessing. For example:
- a `#SELECT#` without a `#CASE#` on its line, or a `#CASE#` with no `#SELECT#` open;
- a malformed `#AUTONUMBER#`;
- a `/*` never closed, or a template string never closed;
- a statement still continuing at the end of the file;
- a `#DEFINE#` with no name, or a name defined twice;
- a reference to a name that was never defined.

brls also tells you where line numbers can't fit. Where a section runs out of numbers, it retries
with a smaller step rather than giving up, and says what it did.

**BR: Show Lexi Expansion** opens the BR a file becomes, read-only, beside it. The expansion
includes BR's line-count markers (`!#-NN`, `!#+NN`) on lines that join several source lines. They
keep BR's line count in step with your source, so the line BR names in an error is the line you
see. Turn them off with `br.lexi.lineMarkers` for output exactly like the original Lexi's.

<a id="the-original-lexi"></a>
### The original Lexi

brls's Lexi fixes defects in the original Lexi program (`lexi.br`). Among them:
- comments hidden by an apostrophe inside a string;
- block comments that never close;
- template strings that find their closing braces past the end of the string;
- select directives acted on inside comments.

**To use the original instead:** install the `crs-dev.vslang-br` extension, **disable** it, then
install this one. Disabled, it neither checks nor colours anything, but its `Lexi` folder stays
installed. Each time the brls extension starts, it looks there. When it finds the folder:

- **Show Lexi Expansion and Compile with BR** run the original `lexi.br` itself, so what you see and
  what BR compiles is the original's output. The original writes no line-count markers. brls still
  traces each BR error to the source lines that produced it.
- **The checks as you type** expand the file the way the original does. That's identical to the
  original's output over a corpus of nearly 3,000 files; running BR on every keystroke would be
  too slow. Lexi source in error is shown as a warning, because the original acts on it.
- **If the original can't run**, Show Lexi Expansion and Compile say so. brls doesn't quietly use
  its own Lexi instead. It runs the original on Windows only.

**BR: Show Output** says which Lexi is in use.

<a id="customizing-lexi"></a>
### Customizing Lexi: the dialect file

Every word Lexi recognises and everything it writes is spelled in one JSON file, the **dialect
file**, `lexi.json`: the `#DEFINE#`, `#SELECT#` and `#AUTONUMBER#` directives, the `&=` append, the
`!_` join, the continuation operators, the backtick string, the block comment, the label form, and
the BR written for each. A project can have its own, so that a team writes Lexi in its own house
spelling.

**Creating one.** Run **BR: Create lexi.json (Lexi dialect) in the Workspace**. It copies the
dialect built into brls into the workspace folder and opens it. An existing `lexi.json` is opened,
never overwritten.

**Where brls looks**, in this order:
1. the file the `br.lexi.dialect` setting names, relative to the workspace folder;
2. `lexi.json` in the workspace folder (the first folder, in a multi-folder workspace);
3. the dialect built into brls.

**What it changes.** Saving the file changes Lexi at once, without a restart. The open Lexi files are
expanded and checked again, and Show Lexi Expansion and Compile with BR use it. The dialect also
decides:
- which files count as Lexi, so a renamed `#SELECT#` marks a file as Lexi and the shipped word no
  longer does;
- the numbers **Add Line Numbers** writes and **Strip Line Numbers** removes;
- the words the Lexi snippets insert.

**The file holds only what you change.** brls reads it on top of the built-in dialect, so a key left
out keeps its built-in value. This is a complete dialect file that makes `<<=` the append operator:

```json
{ "concat": { "find": "<<=" } }
```

**Mistakes are shown on the file.** A file brls can't use is marked with an error at the key, rule
or character it's about, and brls uses its built-in dialect until you fix it. A misspelled key is one
of those errors, rather than being silently ignored.

Things to know:
- **Keep `lexi.json` in version control with your programs.** Compile with BR compiles the
  expansion, so the same source compiles differently under a different dialect file.
- **Choose distinctive words.** A directive that ordinary BR also contains would make ordinary BR
  files count as Lexi.
- **Not with the original Lexi.** With the original Lexi in use, `lexi.json` isn't read; brls says
  so in its output. For the original's expansion from brls's own Lexi instead, a dialect file can
  set `"legacy": true`, `"recordLength": 800` and `"define": {"firstOnly": true}`.
- **Test what you change.** The built-in dialect is held to the original Lexi by brls's tests; a
  dialect of your own isn't. Keep a few files that use each construct you changed, and check their
  expansion after each edit.

---

## Running BR from the editor

These commands run BR. brls decides which BR and which configuration (see
[Telling brls where BR is](#telling-brls-where-br-is)), and runs it in the background, so you can
keep editing while it works.

| Command | Key | What it does |
|---|---|---|
| **BR: Compile with BR** | Ctrl+Shift+B | Saves the file and has BR load it as source and save it compiled, beside the source: `prog.brs` and `prog.br.brs` become `prog.br`, and `.wbs` becomes `.wb`. A Lexi file is expanded first. BR's error, if any, appears on the line it names. Also on the editor tab's context menu. |
| **BR: Run with BR** | Ctrl+Shift+R | Compiles a source file, then starts the program in BR's own window. Also on the context menu of a source or compiled program in the Explorer. |
| **BR: Toggle Compile on Save** | Click the status bar item | Compiles a BR file every time you save it (`br.compile.onSave`). The status bar shows whether it's on. |
| **BR: View Compiled Program** | Explorer context menu on a `.br`, `.bro`, `.wb` or `.wbo` file | Opens BR's listing of a compiled program, read-only. |
| **BR: Decompile to Source** | Explorer context menu on a compiled program | Writes the program's listing to a source file, asking before overwriting one. Off until you set `br.run.allowDecompile`, because a project that keeps its masters as source shouldn't have them overwritten from compiled programs. Viewing a listing is always allowed. |
| **BR: Search Compiled Programs (LIST)** | Ctrl+Alt+F | Runs BR's own `LIST` search over every compiled program in the workspace (see below). |
| **BR: Show BR Runtime Info** | — | Shows BR's `STATUS` reports for the BR brls runs. |

**Searching compiled programs.** It's the only search that sees a program whose source is gone.
- Enter one to three quoted terms: `'text'` matches any case, `"TEXT"` matches exact case, and `~`
  in front of a term excludes lines that contain it.
- Pick a result to open the matching line in the program's source, when the source exists and has
  that line number. Otherwise the program's listing opens.

<a id="telling-brls-where-br-is"></a>
### Telling brls where BR is

These commands need three files and an application root.

| What | Setting | Environment variable | Used by |
|---|---|---|---|
| The BR executable | `br.run.executable` | `BR_EXE` | All of them |
| An unattended configuration (a BRConfig.sys) | `br.run.unattendedConfig` | `BR_AI_UTIL` | Every command but Run |
| An interactive configuration | `br.run.interactiveConfig` | `BR_AI_USER` | Run |

- **Order of precedence:** the setting, then the environment variable, then the copy in the BR
  reference kit, if `br.referenceRoot` names one.
- **All three must exist.** brls finds them together, so even Compile needs the interactive
  configuration to exist.
- **Relative paths** are taken from the application root.
- **The application root** is `APP_ROOT` if set; otherwise the folder containing the kit's
  `context` folder.

So without the kit, set `APP_ROOT` and the three files.

brls runs BR unattended on a copy of your unattended configuration, with its `LOGGING` lines
replaced by the job's own log. Your configuration and its log are never touched.

---

## FileIO layouts

A FileIO layout describes a data file's fields. brls opens a layout as one when it's a `.lay` file,
or sits directly in a folder named in `br.filelay.folders` with the extension in
`br.filelay.extension`.

**In a layout:**
- brls reports problems in it;
- hovering on a field says what FileIO makes of it;
- the fields appear in the Outline.

**Format Document** (Shift+Alt+F) lines the columns up. `br.filelay.format.minWidths` sets the
least width of each of the first five columns. Formatting applies only to layouts. BR source is
never reformatted.

**In BR source:**
- the subscript constants FileIO defines, such as `CO_NAME` for the field `NAME$` of a layout
  prefixed `CO_`, are offered as completions once the word you're typing contains a `_`;
- hovering on one says which field it is.

The extension also starts when a workspace contains a `filelay` folder, so layouts are recognised
before any BR file is opened.

---

## Commands and keys

| Command | Key (when a BR file has focus) |
|---|---|
| BR: Add Line Numbers | Ctrl+Alt+3 |
| BR: Strip Line Numbers | Ctrl+Alt+4 |
| BR: Compile with BR | Ctrl+Shift+B |
| BR: Run with BR | Ctrl+Shift+R |
| BR: Search Compiled Programs (LIST) | Ctrl+Alt+F (anywhere) |
| BR: Show Lexi Expansion | — |
| BR: Create lexi.json (Lexi dialect) in the Workspace | — |
| BR: Toggle Compile on Save | — |
| BR: View Compiled Program | — (Explorer context menu) |
| BR: Decompile to Source | — (Explorer context menu) |
| BR: Show BR Runtime Info | — |
| BR: Restart Language Server | — |
| BR: Show Language Server Output | — |

Rebind any of these in **Keyboard Shortcuts** (Ctrl+K Ctrl+S).

---

## Settings

Every setting is in the `br` section of your settings; search for `br.` in the Settings editor.

| Setting | Default | What it does |
|---|---|---|
| `br.sourceSuffixes` | `[".brs", ".wbs"]` | Which files are source, and how a `LIBRARY` name maps to one |
| `br.closedWorkspace` | `false` | The workspace is the whole project, so a missing `LIBRARY` program is reported |
| `br.maxIndexedFiles` | `20000` | Upper bound on the files indexed at startup |
| `br.ignoreDirectories` | `[".git", ".svn", ".hg", "node_modules", ".vscode", "vendor"]` | Folders not indexed |
| `br.referenceRoot` | found automatically | The folder holding the BR reference (`br_tree/` and `dev/`), for documentation links |
| `br.diagnostics.<key>` | on, except `unused` | Each group of diagnostics (see [Diagnostics](#diagnostics)) |
| `br.autoLineNumbers.enabled` | `true` | Number the line Enter opens |
| `br.autoLineNumbers.increment` | `10` | The step where the lines above give none |
| `br.autoLineNumbers.zeroPadding` | `5` | The number's width where the line above gives none, 1 to 5 |
| `br.snippets.fileio` | `false` | Offer the FileIO snippets |
| `br.lexi.lineMarkers` | `true` | Put BR's line-count markers in Show Lexi Expansion |
| `br.lexi.dialect` | none | The project's Lexi dialect file, relative to the workspace folder. Empty means `lexi.json` in the workspace folder if there is one (see [Customizing Lexi](#customizing-lexi)) |
| `br.run.executable` | `$BR_EXE`, then the kit's | The BR that the run commands start |
| `br.run.unattendedConfig` | `$BR_AI_UTIL`, then the kit's | The configuration BR runs unattended on |
| `br.run.interactiveConfig` | `$BR_AI_USER`, then the kit's | The configuration Run starts the program on |
| `br.run.allowDecompile` | `false` | Let Decompile write a source file |
| `br.compile.onSave` | `false` | Compile each BR file when it's saved |
| `br.filelay.folders` | `["filelay"]` | Folders whose files are FileIO layouts |
| `br.filelay.extension` | none | The extension a layout in those folders has |
| `br.filelay.format.minWidths` | see SETTINGS.md | Least widths of a layout's first five columns when formatted |
| `br.server.path` | the bundled brls | The brls executable to start |
| `br.trace.server` | `"off"` | Record the traffic with brls in its output channel: `off`, `messages` or `verbose` |

[SETTINGS.md](SETTINGS.md) has the full description of each setting and lists every diagnostic rule
under its key.

**Settings are shared with `crs-dev.vslang-br`.** Both extensions use the `br` section, so settings
you made for that one carry over. Its own diagnostic switches (`functions`, `undefinedFunctions`,
`unusedVariables`) mean nothing to brls. brls says so once, rather than treating them as mistakes.
`syntax` is the one key both understand.

A setting brls doesn't recognise is reported once per session in the server's output, not silently
dropped.

---

## The server

The extension starts the brls bundled for your platform. To use another, for example one you built
yourself or one for a platform the package doesn't include, set `br.server.path` to its executable.
The `BRLS_SERVER_PATH` environment variable does the same.

**BR: Restart Language Server** restarts brls. Use it after changing `br.server.path`.
**BR: Show Language Server Output** opens brls's output channel, which shows:
- what brls reported about itself, including settings it couldn't use;
- which Lexi is in use.

Set `br.trace.server` to `messages` or `verbose` to record every message between VS Code and brls
in that channel.

---

## Troubleshooting

| Symptom | Cause and fix |
|---|---|
| Two of every diagnostic, or doubled colouring | `crs-dev.vslang-br` is enabled too. Disable it. |
| "brls has no server for darwin" (or another platform) | The package bundles brls for Windows and Linux only. Set `br.server.path`. |
| Enter doesn't number the next line | `editor.formatOnType` is off for BR files in your settings, or `br.autoLineNumbers.enabled` is off. |
| Box-drawing or accented characters look wrong | The file was opened in another encoding. BR source is CP437, which the extension sets for BR files; check for an `files.encoding` override. |
| Compile or Run says "cannot tell where the application is" | Set `APP_ROOT`, or `br.referenceRoot` to the BR reference kit. See [Telling brls where BR is](#telling-brls-where-br-is). |
| Compile or Run says "no BR executable" or a configuration doesn't exist | Set `br.run.executable`, `br.run.unattendedConfig` and `br.run.interactiveConfig`, or their environment variables. |
| Decompile refuses | Set `br.run.allowDecompile`. |
| "No such library program" never appears | It's only reported when `br.closedWorkspace` is on. |
| Lexi files suddenly full of errors after editing `lexi.json` | The dialect file was refused, and brls fell back to its built-in dialect. The error on `lexi.json` says why. |
| Show Lexi Expansion says the original Lexi can't run | The `crs-dev.vslang-br` extension's `Lexi` folder is incomplete, or you're not on Windows. Uninstall that extension to use brls's own Lexi. |
| Something else | Open **BR: Show Language Server Output**, and set `br.trace.server` to `verbose` to capture the exchange. Report it at [the repository's issues](https://github.com/gordon777dye/BR-vscode-extn-2/issues), with that output. |

---

## License

The extension is licensed by Application Development Systems under the
[Business Rules VS Code Extension License Agreement](LICENSE.md). In short, you may install, use,
copy, and redistribute it unmodified, free of charge, including for commercial work. It doesn't
include BR: the copy of BR the extension runs is yours, under the separate *Business Rules License
Agreement*. The agreement itself is what governs; this summary is not a substitute for it.

The extension includes open-source components under their own licenses, listed in
[THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md).
