# brls settings

<!-- The settings of brls 0.1.4, as the brls extension in this folder reads them. -->

brls reads the `br` section. A client may supply it as `initializationOptions` at the
handshake, in `workspace/didChangeConfiguration`, or by answering brls's `workspace/configuration` request,
which is what VS Code does. The `br` wrapper is optional. The section is shared with other BR tooling, so a key
belonging to another tool is named and ignored.

## Settings

| Setting | Type | Default | What it does |
|---|---|---|---|
| `br.sourceSuffixes` | string[] | `[".brs",".wbs"]` | The ordered chain of source suffixes. A file ending in one is BR source, and a `LIBRARY` name is tried against each. Every entry must begin with a dot; an entry that does not is ignored with a message. |
| `br.closedWorkspace` | boolean | `false` | Say the indexed files are the whole project. Only then is "no such library program" worth reporting; over a folder that is merely open it would fire on every reference to a file outside it. |
| `br.maxIndexedFiles` | integer | `20000` | Bound on the initial scan, so a root chosen by accident (a home directory, a drive root) is a slow start and not an unbounded one. Must be positive. |
| `br.ignoreDirectories` | string[] | `[".git",".svn",".hg","node_modules",".vscode","vendor"]` | Directory names skipped whole during the scan. |
| `br.referenceRoot` | string | found automatically | The directory holding `br_tree/` and `dev/`. Turns each rule's citation into a link the client can open. Empty means brls looks for one nearby, and failing that puts the citation in the message text. It is never guessed. |
| `br.diagnostics` | object | every rule on except `unused` | Switches rules on or off, keyed by the setting names in the second table. A key brls does not know is reported once per session rather than dropped. |
| `br.autoLineNumbers.enabled` | boolean | `true` | Number the line Enter opens, when the line it was pressed on is numbered. Inside a `!:` chain the new line is indented instead. In VS Code this needs `editor.formatOnType`, which the brls extension turns on for BR files. |
| `br.autoLineNumbers.increment` | integer | `10` | The step used when the lines above give none. Otherwise the step is read off the two nearest numbered lines and rounded down to 1, 2, 10, 20 or 100, and further to fit before the next numbered line. |
| `br.autoLineNumbers.zeroPadding` | integer | `5` | The width used when the line being continued gives none. 1 to 5. |
| `br.snippets.fileio` | boolean | `false` | Offer the FileIO snippet group, for projects that link the FileIO library. |
| `br.lexi.lineMarkers` | boolean | `true` | Write BR's line-count markers (`!#-NN`, `!#+NN`) into the BR that *Show Lexi Expansion* produces, on each line that joins source lines or runs past a record, so that BR's line count is the Lexi source's. Turn off for output identical to Lexi's own. |
| `br.lexi.dialect` | string | "" | The Lexi dialect file: the `lexi.json` that says what Lexi recognises and what it writes for it (see `br_tree` LEXI.md, *The dialect file*). A path, relative to the workspace root. Empty means `lexi.json` in the workspace root if there is one, and otherwise the dialect built into brls. The file is read when it is used and again when it changes, so an edit to it is the next expansion's rules. The file is read on top of the built-in dialect, so it need hold only what it changes, and the dialect also decides which files are Lexi, what Add and Strip Line Numbers write, and what the Lexi snippets insert. A file that is not a dialect is an error on the file itself, and brls's own is used until it is fixed; with the original Lexi in use the file is not read. The extension ships a copy to start from: *BR: Create lexi.json*. |
| `br.lexi.original` | string | none: brls's own Lexi | *(`initializationOptions` only)* The folder of the original Lexi (`lexi.br`, `linenum.brs`, `lexionly.brs`, `wbconfig.sys` and `brnative.exe`). Set, *Show Lexi Expansion* and *Compile* run `lexi.br` itself, on Windows only, and the checks expand as it does. Empty means brls's own Lexi. The brls VS Code extension sets it to the `Lexi` folder of an installed, disabled `crs-dev.vslang-br` extension. Read at the handshake only. |
| `br.run.executable` | string | `$BR_EXE`, then the kit's | The BR executable that *Compile*, *Run*, *Decompile*, *BR Runtime Info* and *Search Compiled Programs* start. Empty means `$BR_EXE`, then the kit's `dev/tools/brserver-x64-Release.exe`. A relative path is taken from the application root, the folder holding `context/`. |
| `br.run.unattendedConfig` | string | `$BR_AI_UTIL`, then the kit's | The configuration BR runs unattended on, for every command but *Run*. Empty means `$BR_AI_UTIL`, then the kit's `dev/tools/brconfig.ai_util`. brls runs BR on a copy written beside it, with its `LOGGING` lines replaced by the job's own unattended log, so `startlog.txt` is never touched. |
| `br.run.interactiveConfig` | string | `$BR_AI_USER`, then the kit's | The configuration *Run* starts the program on, in BR's own window. Empty means `$BR_AI_USER`, then the kit's `dev/tools/brconfig.ai_user`. |
| `br.run.allowDecompile` | boolean | `false` | Let *Decompile* write a source file. Off by default, because where a project keeps its program masters as source the kit forbids decompiling (ONBOARDING STEP 2). Viewing a compiled program's listing is always allowed: it writes nothing. |
| `br.compile.onSave` | boolean | `false` | Compile a BR source file with BR each time it is saved. BR's verdict appears as a diagnostic on the file until it is next edited. |
| `br.filelay.folders` | string[] | `["filelay"]` | Folder names whose files are FileIO layouts (`Appendix A` of the kit's ONBOARDING.md), besides any file ending `.lay`. Their `version` and `template` subfolders are not layouts in use and are not indexed. |
| `br.filelay.extension` | string | "" | The extension a layout in those folders has: FileIO's `layoutextension` setting in fileio.ini, which is none unless set. A file there with another extension is not a layout. |
| `br.filelay.format.minWidths` | integer[] | `[24,26,8,0,10]` | The least widths of a layout's columns 1 to 5 (name, description, FORM, date, positions) when it is formatted, each counting its comma and the spaces after it. A column is the greater of this and its widest entry with its comma and two spaces. The defaults are the commonest widths in QSMRP's layouts. |
| `br.server.path` | string | the bundled brls | *(VS Code extension)* The brls executable the VS Code extension starts. Empty means the one bundled with the extension. Read by the extension, not by brls. |
| `br.trace.server` | string | `"off"` | *(VS Code extension)* Records the LSP traffic in the extension's output channel: `off`, `messages` or `verbose`. Read by the extension, not by brls. |

## Diagnostic toggles

Each is `br.diagnostics.<key>`, a boolean. One key can govern several rules.

| Key | Default | Rules it governs |
|---|---|---|
| `concatTypeMismatch` | on | `numeric-in-concat` |
| `defLibraryComma` | on | `def-library-stray-comma` |
| `executeStrings` | on | `execute-statement-syntax`, `execute-command-syntax`, `execute-ends-program`, `execute-free-abbreviated` |
| `fileioSubscripts` | on | `unknown-subscript` |
| `formats` | on | `bad-using-format`, `using-not-a-form` |
| `libraries` | on | `not-exported`, `library-function-not-defined`, `unresolved-library` |
| `mixedAndOr` | on | `mixed-and-or` |
| `notAnOperator` | on | `ignored-argument`, `not-an-operator`, `evaluates-to-zero` |
| `reservedNames` | on | `reserved-name`, `reserved-io-target`, `intrinsic-array-name` |
| `shadowsIntrinsic` | on | `shadows-intrinsic` |
| `subscriptTypeMismatch` | on | `string-subscript` |
| `surprisingGrouping` | on | `exponent-associativity`, `negation-binds-tighter` |
| `syntax` | on | `syntax-error`, `line-too-long`, `duplicate-line-number`, `line-number-order`, `duplicate-option`, `continuation-into-numbered-line`, `bare-label`, `empty-parens-call`, `empty-parens-def`, `bad-assignment-target`, `non-functional-form` |
| `unclosedBlocks` | on | `unclosed-for`, `unmatched-next`, `unclosed-do`, `unmatched-loop`, `unclosed-if`, `nested-if` |
| `undefinedTargets` | on | `undefined-target`, `duplicate-target` |
| `unknownFunctions` | on | `unknown-function`, `unimplemented-function`, `deprecated-function`, `duplicate-function`, `duplicate-parameter`, `redim-unpassed-optional-array`, `unclosed-def`, `intrinsic-missing-arguments`, `intrinsic-arguments`, `missing-argument`, `literal-by-reference`, `argument-type`, `wrong-argument-count`, `library-name-not-a-function` |
| `unused` | off | `unused-library`, `unused-parameter`, `unused-dim` |
