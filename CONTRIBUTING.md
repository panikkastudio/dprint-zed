# Contributing

This repository contains a Zed extension written in Rust (compiled to WASM). The extension registers
a language server with id `dprint` and tells Zed how to launch the `dprint` CLI in **LSP mode**.

## Ground truth (match the code)

### What the extension does

- Exposes a Zed language server with id **`dprint`** (defined in `extension.toml`).
- When Zed requests the language server command, the extension returns:
  - a `command` (path to a `dprint` executable), and
  - `args` (defaults to `["lsp"]` unless overridden in Zed settings).
- Resolves the `dprint` executable in this order:
  1. `lsp.dprint.binary.path` from Zed settings (if set).
  2. Worktree `node_modules/.bin/dprint` **only if**:
     - the worktree indicates `dprint` is used (via `package.json` deps/devDeps or `deno.json`
       imports), and
     - `node_modules/.bin/dprint` exists.
  3. `dprint` found on `PATH`.
  4. Auto-download the latest stable `dprint` release from `dprint/dprint` GitHub releases and run
     it.

### What the extension does NOT do

- It does **not** implement formatting logic itself.
- It does **not** run `dprint fmt` as a fallback.
- It does **not** forward custom LSP “settings” payloads to `dprint`.
  - If you see docs mentioning settings like `config_path`, `require_config_file`,
    `requireConfiguration`, or `configurationPath`: that behavior is **not implemented** in this
    codebase.
- Auto-install does **not** support 32-bit `x86` architecture (manual binary configuration
  required).

## Development setup

### Prerequisites

- Rust toolchain (stable is fine; this repo uses Rust edition 2024).
- WASM target for Zed extensions:

```
rustup target add wasm32-wasip2
```

### Build / check locally

From the repo root:

```
cargo check
```

(Optional, if you use them locally:)

```
cargo fmt
cargo clippy
```

## Running the extension in Zed (dev install)

1. Open Zed.
2. Command palette → `zed: install dev extensions`.
3. Select the repository folder.
4. After changes, open `zed: extensions`, find the extension, and click **Rebuild**.

## Testing behavior (truthy workflow)

### 1) Validate the command Zed will run

By default the extension runs:

- `dprint lsp`

If you override arguments, it will run whatever you set in `lsp.dprint.binary.arguments`.

To reduce ambiguity while testing, explicitly pin the binary path in your Zed `settings.json`:

```
{
  "lsp": {
    "dprint": {
      "binary": {
        "path": "/absolute/path/to/dprint",
        "arguments": ["lsp"]
      }
    }
  }
}
```

Notes:

- If you omit `arguments`, the extension defaults to `["lsp"]`.
- If you’re using a wrapper script/binary, point `path` at the wrapper and set `arguments`
  accordingly.

### 2) Test the worktree `node_modules/.bin` detection (optional)

The extension will only prefer `node_modules/.bin/dprint` if the worktree declares `dprint` as being
used.

For Node projects:

- Ensure `package.json` has `dprint` in `dependencies` or `devDependencies`.
- Ensure `node_modules/.bin/dprint` exists (for example after install).

For Deno projects:

- Ensure `deno.json` has `dprint` under `imports`.

Then start Zed in that worktree and confirm (via logs) that it launches the worktree binary.

### 3) Test the auto-installer path (optional)

To exercise auto-install:

- Ensure there is no `lsp.dprint.binary.path` override.
- Ensure the worktree does not qualify for `node_modules/.bin/dprint` selection.
- Ensure `dprint` is not available on `PATH`.

When Zed starts the language server, the extension should download the latest stable `dprint` from
GitHub releases and run it.

Caveats:

- Auto-install is OS/architecture dependent.
- 32-bit `x86` is not supported by the auto-installer.

### 4) Confirm dprint formatting behavior independently

If the language server starts but formatting output is unexpected, validate `dprint` itself in the
same repository/worktree context:

- `dprint --version`
- `dprint fmt`

Configuration discovery is handled by `dprint` (not by this extension).

## Repository conventions

- Keep changes small and focused.
- Prefer clear error messages and deterministic behavior.
- Update `README.md` / `CHANGELOG.md` when behavior changes.

## Filing issues / PR testing notes

When you open an issue or PR, include:

- Zed version
- OS + architecture
- The resolved `dprint` binary path (and whether it was auto-installed, from PATH, or from
  `node_modules`)
- The args used (default `["lsp"]` or overridden)
- `dprint --version` output
- Relevant Zed logs mentioning `dprint`

## Security

Do not commit secrets (tokens, API keys, credentials). If you add tooling that needs credentials,
document configuration via environment variables or local-only config files.
