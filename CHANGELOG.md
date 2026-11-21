# Changelog

Each release is on the repository's
[Releases page](https://github.com/gordon777dye/BR-vscode-extn-2/releases), with its `.vsix`.

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
