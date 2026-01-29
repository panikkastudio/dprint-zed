# Dprint (dprint) for Zed

This repository contains a Zed extension that wires Zed’s Language Server support to the
**`dprint`** CLI running in **LSP mode**.

The extension **does not implement formatting itself**. It launches `dprint lsp` (by default) and
lets Zed talk to it via the Language Server Protocol.

---

## How it works

### Language server id

The language server id is **`dprint`** (as defined in `extension.toml`). Any Zed `lsp` configuration
you apply must be under:

- `lsp.dprint`

### Binary resolution order (truthy to the code)

When Zed asks the extension to start the language server, the extension resolves which `dprint`
executable to run in this order:

1. **Zed settings override**: if you configured `lsp.dprint.binary.path`, that path is used.
2. **Workspace `node_modules` install**: if your worktree indicates you use `dprint` and
   `node_modules/.bin/dprint` exists, that path is used.
   - “Indicates you use `dprint`” means:
     - `package.json` contains `dprint` in `dependencies` or `devDependencies`, **or**
     - `deno.json` contains `dprint` in `imports`
   - Be sure to run `npm install` or equivalent to install `dprint` in `node_modules`. As Zed
     extension host does not have the permission to check if a binary is there.
3. **System `PATH`**: if `dprint` is found via `which`, it’s used.
4. **Auto-install fallback**: if none of the above apply, the extension downloads the latest stable
   release from `dprint/dprint` GitHub releases and runs it.

Notes:

- The auto-installed binary is downloaded as a GitHub release zip appropriate for your OS + CPU
  architecture.
- `x86` (32-bit) is **not supported** by the auto-installer; in that case you must provide a
  `dprint` binary yourself.

### Arguments

- If you set `lsp.dprint.binary.arguments`, those arguments are used.
- Otherwise the extension starts dprint with:

- `["lsp"]`

So by default it runs:

- `dprint lsp`

---

## Supported languages

Zed will attempt to use this language server for the languages declared in `extension.toml`:

- JavaScript
- TypeScript
- TSX
- JSON
- JSONC
- Markdown
- TOML
- CSS
- SCSS
- LESS
- HTML
- Vue.js
- Svelte
- Astro
- Angular
- Twig
- GraphQL
- YAML
- PHP
- Python

Whether formatting works for a given file still depends on your `dprint` configuration and plugins.

---

## Configuration

### 1) Recommended: pin the `dprint` binary Zed should run

When troubleshooting, explicitly point Zed at the binary and arguments you want.

Example `settings.json`:

```json
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

If you omit `arguments`, the extension defaults to `["lsp"]`.

### 2) “Extension settings” passed to the server

This extension currently **does not add or forward custom LSP settings** to `dprint`.

If you see examples referencing settings like `config_path`, `require_config_file`,
`requireConfiguration`, or `configurationPath`, those are **not implemented by this codebase**.

What actually happens today:

- The extension only provides Zed a command + args (and no extra environment variables).
- `dprint` discovers configuration the same way it normally does when you run it (for example, by
  finding `dprint.json` / `.dprint.json` according to `dprint`’s own rules).

If you need non-default config discovery behavior, you currently must accomplish that via:

- running `dprint` from an environment where it can find the desired config, or
- using a wrapper script/binary and configuring `lsp.dprint.binary.path` to point to that wrapper
  (advanced).

---

## Auto-install behavior (details)

If the extension auto-installs `dprint`, it:

- checks the latest stable GitHub release from `dprint/dprint`
- downloads the matching zip asset for your platform
- removes older `dprint-*` directories/files in the extension working directory
- runs the downloaded `dprint` binary

This is meant as a convenience fallback so the extension can work even when `dprint` is not already
installed.

---

## Troubleshooting

### The server fails to start

Most common causes:

- The configured binary isn’t executable or isn’t the `dprint` you expect.
- The binary arguments aren’t correct (if you overrode them).
- You’re on 32-bit (`x86`) and relying on auto-install (unsupported).

Quick checks:

- Run the exact command Zed would run (for example `dprint lsp`) in a terminal.
- Temporarily set `lsp.dprint.binary.path` to a known-good `dprint` and keep arguments as `["lsp"]`.

### Formatting doesn’t change files

If the language server starts but formatting seems ineffective, validate your `dprint` configuration
and plugins by running `dprint fmt` manually in the same worktree and ensuring it produces the
expected output.

---

## Development

### Prerequisites

- Rust toolchain (stable is fine)
- `wasm32-wasip2` target (Zed loads Rust extensions as WASM):

```sh
rustup target add wasm32-wasip2
```

### Build/check

```sh
cargo check
```

### Load as a dev extension in Zed

1. Open Zed
2. Run `zed: install dev extensions`
3. Select the repository directory
4. After changes: open `zed: extensions`, find the extension, and click **Rebuild**

---

## License

MIT
