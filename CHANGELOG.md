# Changelog

All notable changes to this extension are documented in this file.

This project follows a pragmatic changelog format. Versions listed here refer to the Zed extension
package version in `extension.toml`.

## 0.1.0

### Added

- Initial public release of the Zed extension.
- Registers a language server with id `dprint` for the languages listed in `extension.toml`.

### Implemented

- Language server command construction that runs `dprint` in LSP mode (defaults to `dprint lsp`).
- Binary resolution order:
  1. Uses `lsp.dprint.binary.path` from Zed settings if provided.
  2. Uses workspace `node_modules/.bin/dprint` when:
     - the worktree declares `dprint` in `package.json` (`dependencies` or `devDependencies`) or
       `deno.json` (`imports`), and
     - `node_modules/.bin/dprint` exists.
  3. Falls back to `dprint` found on `PATH`.
  4. Otherwise auto-downloads the latest stable `dprint` release from `dprint/dprint` GitHub
     releases and runs it.
- Auto-installer behavior:
  - Downloads the correct OS/architecture zip asset.
  - Removes previously downloaded `dprint-*` release folders/files before installing the new
    version.
  - Does not support 32-bit `x86` auto-install (manual binary configuration required on that
    architecture).
