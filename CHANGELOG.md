# Changelog

Each release is on the repository's
[Releases page](https://github.com/gordon777dye/BR-vscode-extn-2/releases), with its `.vsix`.

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
